Compose通过以下方法把State转成了UI:

```
State-> Composition(组合) -> Layout (布局) -> Drawing (绘制) -> UI
```

本文章讲述`Layout`部分，一些代码块来帮助我们对于UI元素的布局。

一、目标：

高性能：只测量一次子项。能更高效的遍历更深的界面树。以前的View系统就是O(n方)。这是通过固有特性测量实现的。

自定义layouts：有别的文章介绍。

二、基本

示例，就是一个Composable函数，可以包含多个界面元素，但是他们堆叠在一起了。

```''
@Composable
fun ArtistCard() {
    Text("Alfred Sisley")
    Text("3 minutes ago")
}
```

todo补图。

因此Compose提供了一些标准的布局来排列元素。大部分情况只使用标准布局元素（layout包下的类）就好了。

比如：

```kotlin
@Composable
fun ArtistCardColumn() {
    Column { //追加了Column垂直布局，就让他们竖向往下排列显示；
        Text("Alfred Sisley")
        Text("3 minutes ago")
    }
}

@Composable
fun ArtistCardRow(artist: Artist) {
    Row(verticalAlignment = Alignment.CenterVertically) { //使用Row，就可以水平的排列显示；
        Image(bitmap = artist.image, contentDescription = "Artist image")
        Column {
            Text(artist.name)
            Text(artist.lastSeenOnline)
        }
    }
}

@Composable
fun ArtistAvatar(artist: Artist) {
    Box { //使用 Box 可将元素放在其他元素上。Box 还支持为其包含的元素配置特定的对齐方式。
        Image(bitmap = artist.image, contentDescription = "Artist image")
        Icon(Icons.Filled.Check, contentDescription = "Check mark")
    }
}
```

todo补图。

还可以通过设置Row的子属性，调整位置:

```kotlin
Row(
      verticalAlignment = Alignment.CenterVertically,
      horizontalArrangement = Arrangement.End
  ) {
      Image(bitmap = artist.image, contentDescription = "Artist image")
      Column { /*...*/ }
  }
```

Compose 可以有效地处理嵌套布局，不用害怕嵌套！

三、布局模型

父节点会在其子节点之前进行测量，但会在其子节点的尺寸和放置位置确定之后再对自身进行调整。compose通过一次测量子项，避免多次测量O(n2)的性能问题。

后续章节：Compose固有特性测量。

四、修饰符（Modifiers）

```kotlin
@Composable
fun ArtistCardModifiers(
    artist: Artist,
    onClick: () -> Unit
) {
    val padding = 16.dp
    Column(
        Modifier
            .clickable(onClick = onClick)
            .padding(padding)
            .fillMaxWidth()
    ) {
        Row(verticalAlignment = Alignment.CenterVertically) { /*...*/ }
        Spacer(Modifier.size(padding))
        Card(
            elevation = CardDefaults.cardElevation(defaultElevation = 4.dp),
        ) { /*...*/ }
    }
}
```

在上面的代码中，结合使用了不同的修饰符函数。

- `clickable` 使可组合项响应用户输入，并显示涟漪。
- `padding` 在元素周围留出空间。
- `fillMaxWidth` 使可组合项填充其父项为它提供的最大宽度。
- `size()` 指定元素的首选宽度和高度。



五、自适应布局

考虑不同屏幕尺寸，根据屏幕配置调整布局。

约束条件：BoxWithConstraints。