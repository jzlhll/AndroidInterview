EarlyEntryPoints.get(Utils.getApp(),  BaseComponent::class.java).getStorage()



@EarlyEntryPoint
@InstallIn(SingletonComponent::class)
interface BaseComponent {

    fun getStorage():Storage
    
    fun getAnalytics() : Analytics
}



   private val storage: Storage = EarlyEntryPoints.get(Utils.getApp(), BaseComponent::class.java).getStorage()





package cn.mycode.mercury.gallery.transfer.task

import cn.mycode.mercury.gallery.transfer.TransferStateMachine
import com.tinder.StateMachine
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import java.util.UUID

abstract class TransferTask(val runnable: Runnable ? = null) {

    val id = UUID.randomUUID().toString()
    
    protected val _taskStateFlow = MutableStateFlow<State>(State.Idle)
    val taskStateFlow = _taskStateFlow.asStateFlow()
    
    var count : Int = 0
    
    open fun onWaiting() {
        _taskStateFlow.value = State.Waiting
    }
    
    fun onStarted() {
        _taskStateFlow.value = State.Started
    }
    
    abstract fun onConnectStateChange(state: TransferStateMachine.State)
    
    open fun onConnectTransitionChange(transition: StateMachine.Transition.Valid<TransferStateMachine.State, TransferStateMachine.Event, TransferStateMachine.SideEffect>) {
    
    }
    
    open fun onClear() {}
    
    open fun execute() {
        runnable?.run()
    }
    
    fun onComplete() {
        _taskStateFlow.value = State.Completed
        onClear()
    }
    
    protected fun onError() {
        _taskStateFlow.value = State.Error
        onClear()
    }
    
    fun onPause() {
        _taskStateFlow.value = State.Paused
    }
    
    open fun onAborted() {
        _taskStateFlow.value = State.Aborted
        onClear()
    }
    
    fun isActive() = _taskStateFlow.value is State.Started || _taskStateFlow.value is State.Waiting
    
    override fun toString(): String {
        return "TransferTask(id=$id, state=${_taskStateFlow.value::class.simpleName})"
    }
    
    sealed class State {
        data object Idle : State()
        data object Waiting : State()
        data object Started : State()
        data object Paused : State()
        data object Completed : State()
        data object Aborted : State()
        data object Error : State()
    }
}













package cn.mycode.mercury.gallery.x3.transfertask

import cn.mycode.mercury.connectivity.ConnectStateMachine
import cn.mycode.mercury.common.base.BaseComponent
import cn.mycode.mercury.gallery.process.track.GalleryTrackUtils
import cn.mycode.mercury.gallery.transfer.TransferStateMachine
import cn.mycode.mercury.gallery.transfer.task.TransferTask
import cn.mycode.mercury.gallery.transfer.task.TransferTaskManager
import cn.mycode.mercury.gallery.x3.GalleryViewModel.ImportViewState
import com.blankj.utilcode.util.Utils
import com.ffalcon.analytics.Analytics
import com.ffalcon.storage.Storage
import com.tinder.StateMachine
import dagger.hilt.android.EarlyEntryPoints
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.flow.filterNotNull
import timber.log.Timber

internal abstract class GalleryTransferTask : TransferTask(), MediaDetector {

    protected val viewState = MutableStateFlow<ImportViewState?>(null)
    val importViewState = viewState.asStateFlow().filterNotNull()
    
    private val storage: Storage = EarlyEntryPoints.get(Utils.getApp(), BaseComponent::class.java).getStorage()
    private val appAnalytics: Analytics =
        EarlyEntryPoints.get(Utils.getApp(), BaseComponent::class.java).getAnalytics()
    
    override fun onWaiting() {
        super.onWaiting()
        viewState.value = ImportViewState.Waiting
    }
    
    override fun onConnectStateChange(state: TransferStateMachine.State) {
    }
    
    override fun onConnectTransitionChange(transition: StateMachine.Transition.Valid<TransferStateMachine.State, TransferStateMachine.Event, TransferStateMachine.SideEffect>) {
        Timber.i("onConnectTransitionChange:${transition.toState}, event:${transition.event}")
        if (transition.toState is TransferStateMachine.State.Idle
            && transition.event !is TransferStateMachine.Event.Stop
        ) {
            val cause = if (transition.event == TransferStateMachine.Event.LanConnectFailed) {
                ImportViewState.ImportError(cause = ImportViewState.ImportError.ERROR_BY_LAN_CONNECTED_FAIL)
            } else if (transition.event == TransferStateMachine.Event.NotAllReady("bleDisconnected")){
                ImportViewState.ImportError(cause = ImportViewState.ImportError.ERROR_BY_BT)
            } else {
                ImportViewState.ImportError(cause = ImportViewState.ImportError.ERROR_BY_UNKNOWN)
            }
            viewState.value = cause
            GalleryTrackUtils.reportTransferLastEvent(appAnalytics, storage)
            return
        }
        if (transition.toState !is TransferStateMachine.State.Idle) {
            val subMsg = if (transition.toState == TransferStateMachine.State.ConnectingP2p) {
                if (transition.event == TransferStateMachine.Event.JmdnsConnect) {
                    TransferStateMachine.SubMsg.JmdnsConnecting
                } else {
                    TransferStateMachine.SubMsg.P2pConnecting
                }
            } else null
            viewState.value = ImportViewState.Connecting(transition.toState, subMsg)
        }
    }
    
    fun cancel() {
        TransferTaskManager.cancel(id)
        _taskStateFlow.value = State.Idle
    }
    
    fun retry() {
        TransferTaskManager.retry()
    }
    
    abstract fun resetViewState()
    
    var onPendingMediasReady: (() -> Unit)? = null

}













package cn.mycode.mercury.gallery.x3.transfertask

import cn.mycode.mercury.common.AppModuleHandle
import cn.mycode.mercury.common.di.ApplicationScope
import cn.mycode.mercury.common.pipeline.Payload
import cn.mycode.mercury.connectivity.device.DeviceManager
import cn.mycode.mercury.connectivity.device.data.Device
import cn.mycode.mercury.gallery.database.room.dao.PhotosDao
import cn.mycode.mercury.gallery.transfer.FileUtil
import cn.mycode.mercury.gallery.transfer.TransferSessionManager
import cn.mycode.mercury.gallery.x3.BluetoothMediator
import cn.mycode.mercury.gallery.x3.GalleryViewModel.ImportViewState
import cn.mycode.mercury.gallery.x3.SyncData
import cn.mycode.mercury.gallery.x3.data.GallerySyncHelper
import cn.mycode.mercury.gallery.x3.data.toMediaItem
import com.blankj.utilcode.util.GsonUtils
import com.google.gson.Gson
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.Job
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.filter
import kotlinx.coroutines.flow.filterNotNull
import kotlinx.coroutines.flow.flatMapLatest
import kotlinx.coroutines.flow.launchIn
import kotlinx.coroutines.flow.map
import kotlinx.coroutines.flow.onEach
import kotlinx.coroutines.flow.onStart
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.flow.take
import kotlinx.coroutines.launch
import timber.log.Timber
import java.util.Collections
import javax.inject.Inject
import kotlin.time.Duration.Companion.seconds

@OptIn(ExperimentalCoroutinesApi::class)
internal class MercuryGallerySyncTask @Inject constructor(
    @ApplicationScope private val scope: CoroutineScope,
    private val sessionManager: TransferSessionManager,
    private val bluetoothMediator: BluetoothMediator,
    private val photosDao: PhotosDao,
    private val gson: Gson,
    private val deviceManager: DeviceManager
) : GalleryTransferTask() {

    private var jobSession: Job? = null
    
    private var mainJob: Job? = null
    private var checkSupportNewJob: Job? = null
    private val _supportsNew = MutableStateFlow(false)
    
    private var mediaCount = MutableStateFlow<Int>(0)
    private val mediaList = MutableStateFlow(emptyList<Int>())
    private val successMedias = Collections.synchronizedSet(mutableSetOf<Int>())
    
    override fun onWaiting() {
        super.onWaiting()
        viewState.value = ImportViewState.Waiting
    }
    
    override fun execute() {
        Timber.i("execute")
        jobSession?.cancel()
        successMedias.clear()
        sessionManager.observeASyncedMediaItems()
            .onStart { sessionManager.onTransfer(FileUtil.Q_SYNC, null, null) }
            .flatMapLatest {
                Timber.d("sync completed, local ids: $it")
                photosDao.getPhotosFlowByLocalIds(it).take(1)
            }
            .filterNotNull()
            .flatMapLatest {
                GallerySyncHelper.progressState.onStart {
                    GallerySyncHelper.batchSyncOriginal(it.map { it.toMediaItem() }.sortedBy { it.dateTaken })
                }
            }
            .onEach {
                Timber.d("Update progress: $it")
                if (it != null) {
                    if (viewState.value is ImportViewState.ImportError) {
                        viewState.value = (viewState.value as ImportViewState.ImportError).copy(
                            progress = ImportViewState.ProgressModel.from(it)
                        )
                    } else {
                        viewState.value =
                            ImportViewState.Importing(ImportViewState.ProgressModel.from(it))
                    }
                    bluetoothMediator.pushBlePayload(
                        Payload(
                            cmd = BLE_CMD_DEL_REMOTE,
                            data = GsonUtils.toJson(it.successIds)
                        )
                    )
                }
                if (it?.isCompleted() == true) {
                    Timber.d("Import completed..")
                    viewState.value = ImportViewState.ImportCompleted(it.total)
                    mediaList.value -= successMedias
                    delay(5000)
                    viewState.value = ImportViewState.None
                    onComplete()
                }
            }
            .launchIn(scope)
            .let { jobSession = it }
    }
    
    override val supportsNew = _supportsNew
    
    override val pendingCount: StateFlow<Int> = mediaCount
    
    override fun startObserve() {
        mainJob?.cancel()
        bluetoothMediator.connectState
            .onEach {
                Timber.d("Bluetooth connect state changed: $it")
                if (it !is AppModuleHandle.ConnectState.Connected) {
                    cancelTimerForCheckIfSupportNew()
                }
            }
            .filter { it is AppModuleHandle.ConnectState.Connected }
            .flatMapLatest {
                bluetoothMediator.observeBlePayload().onStart {
                    checkRemoteUpdates()
                    Timber.d("pushed pre sync cmd")
                    if (deviceManager.currentDevice.value?.type == Device.Type.X2) {
                        timerForCheckIfSupportNew()
                    } else {
                        _supportsNew.value = true
                        cancelTimerForCheckIfSupportNew()
                    }
                }
            }
            .filter { it.cmd == BLE_CMD_PRE_SYNC }
            .map { payload ->
                Timber.d("New items: ${payload.data}")
                cancelTimerForCheckIfSupportNew()
                _supportsNew.value = true
                gson.fromJson<List<SyncData>>(
                    payload.data!!,
                    GsonUtils.getListType(SyncData::class.java)
                ).map { data -> data.id.toInt() }
            }
            .onEach {
                mediaList.value = it
                mediaCount.value = it.size
                successMedias.clear()
            }
            .launchIn(scope)
            .let { mainJob = it }
    }
    
    override fun checkRemoteUpdates() {
        bluetoothMediator.pushBlePayload(Payload(cmd = BLE_CMD_PRE_SYNC))
    }
    
    override fun getRemoteCount() {
        bluetoothMediator.pushBlePayload(Payload(cmd = BLE_CMD_GET_FILES_COUNT))
    }
    
    override fun cleanup() {
        jobSession?.cancel()
        mainJob?.cancel()
        checkSupportNewJob?.cancel()
        successMedias.clear()
    }
    
    override fun resetViewState() {
        if (viewState.value is ImportViewState.ImportAbort
            || viewState.value is ImportViewState.ImportError
        ) {
            viewState.value = if (pendingCount.value > 0) {
                ImportViewState.HasPendingItems(pendingCount.value)
            } else if (supportsNew.value) {
                ImportViewState.None
            } else {
                ImportViewState.StartForLegacy
            }
        }
    }
    
    private fun timerForCheckIfSupportNew() {
        cancelTimerForCheckIfSupportNew()
        scope.launch {
            Timber.d("Start timer for check if support new")
            delay(5.seconds)
            _supportsNew.value = false
        }.let { checkSupportNewJob = it }
    }
    
    private fun cancelTimerForCheckIfSupportNew() {
        Timber.d("cancelTimerForCheckIfSupportNew")
        checkSupportNewJob?.cancel()
    }
    
    override fun onClear() {
        super.onClear()
        jobSession?.cancel()
        successMedias.clear()
    }


    companion object {
        internal const val BLE_CMD_PRE_SYNC = "pre_sync"
        internal const val BLE_CMD_PRE_SYNC_CONFIRMED = "pre_sync_confirm"
        internal const val BLE_CMD_DEL_REMOTE = "delete_remote"
        internal const val BLE_CMD_GET_FILES_COUNT = "get_files_count"
    }
}











package cn.mycode.mercury.gallery.x3

import cn.mycode.mercury.common.AppModuleHandle
import cn.mycode.mercury.common.pipeline.Payload
import cn.mycode.mercury.connectivity.BleRequestPipeline
import cn.mycode.mercury.connectivity.BleResponsePipeline
import cn.mycode.mercury.connectivity.GalleryRequest
import cn.mycode.mercury.connectivity.GalleryResponse
import cn.mycode.mercury.connectivity.observe
import kotlinx.coroutines.flow.map
import timber.log.Timber
import javax.inject.Inject
import javax.inject.Singleton

@Singleton
class BluetoothMediator @Inject constructor(appModuleHandle: AppModuleHandle) {

    val connectState = appModuleHandle.btConnectState
    
    fun observeBlePayload() = BleRequestPipeline.observe<GalleryRequest>().map { it.payload }
    
    fun pushBlePayload(payload: Payload) {
        Timber.d("pushBlePayload($payload)")
        BleResponsePipeline.push(GalleryResponse(payload))
    }
}











inline fun <reified T : BleRequest> BleRequestPipeline.observe(): Flow<T> {
    return observe().filter { it is T }.map { it as T }
}





​    private val requests = MutableSharedFlow<BleRequest>(extraBufferCapacity = 20)

