# Compose Instant
> I might forget how to implement this

## Easy Collapsing Toolbar

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun Screen() {
    val scrollBehavior = TopAppBarDefaults.enterAlwaysScrollBehavior() // (1)

    Scaffold(
        modifier = Modifier
                   .nestedScroll(scrollBehavior.nestedScrollConnection), // (2)
        floatingActionButton = {
           // FAB
        },
        topBar = {
            TopAppBar(
                title = {
                    Text(text = "My APP", style = MaterialTheme.typography.titleLarge)
                },
                scrollBehavior = scrollBehavior, // (3)
                )
        }
    ) {
       // CONTENT
    }
}
```
```gradle
    implementation 'androidx.compose.material3:material3'
```

## Swipe to dismiss

🔗 [Refrence Article](https://medium.com/mobile-app-development-publication/jetpack-compose-swipe-to-dismiss-made-easy-323ca80a0355)

```kotlin
@OptIn(ExperimentalMaterial3Api::class, ExperimentalFoundationApi::class)
inline fun <T> LazyListScope.swipeableItems(
    items: List<T>,
    modifier: Modifier = Modifier,
    noinline key: ((item: T) -> Any)? = null,
    noinline onStateChange: (state: DismissValue, item: T) -> Boolean,
    crossinline background: @Composable() (RowScope.(dismissState: DismissState) -> Unit) = {},
    noinline contentType: (item: T) -> Any? = { null },
    crossinline itemContent: @Composable LazyItemScope.(item: T) -> Unit,
) {
    items(
        items = items,
        key = key,
        contentType = contentType,
    ) { item ->
        val currentItem by rememberUpdatedState(newValue = item)
        val dismissState = rememberDismissState(
            confirmValueChange = { state ->
                onStateChange(state, currentItem)
            }
        )

        SwipeToDismiss(
            state = dismissState,
            background = {
                background(dismissState)
            },
            dismissContent = {
                itemContent(this@items, item)
            },
            modifier = modifier.animateItemPlacement(),
        )

    }
}
```

Usage:
```kotlin 
    LazyColumn {
        swipeableItems(
            items = accounts,
            key = { it.id },
            onStateChange = { state, item ->
                false
            },
            background = {state ->
               // background
            }
        ) { item ->
            AccountItem(item, onAccountClick)
        }
    }
```

```gradle
    implementation 'androidx.compose.material3:material3:1.2.0-alpha02'
```

---

To quickly add background:
```kotlin

@Composable
@OptIn(ExperimentalMaterial3Api::class)
fun EasySwipeableBackground(
    modifier: Modifier = Modifier,
    scaleOnDismiss: Float = 1.5f,
    dismissState: DismissState,
    backgrounds: SwipeableItemBackgrounds,
    startIcon: SwipeableItemIcon,
    endIcon: SwipeableItemIcon
) {

    val direction = dismissState.dismissDirection ?: return

    val backgroundColor by animateColorAsState(
        when (dismissState.targetValue) {
            DismissValue.Default -> backgrounds.defaultColor
            DismissValue.DismissedToEnd -> backgrounds.dismissedToEndColor
            DismissValue.DismissedToStart -> backgrounds.dismissedToStartColor
        }
    )

    val startIconColor by animateColorAsState(
        when (dismissState.targetValue) {
            DismissValue.Default -> startIcon.defaultColor
            DismissValue.DismissedToEnd -> startIcon.dismissedColor
            DismissValue.DismissedToStart -> startIcon.dismissedColor
        }
    )

    val endIconColor by animateColorAsState(
        when (dismissState.targetValue) {
            DismissValue.Default -> endIcon.defaultColor
            DismissValue.DismissedToEnd -> endIcon.dismissedColor
            DismissValue.DismissedToStart -> endIcon.dismissedColor
        }
    )

    val iconColor = when (direction) {
        DismissDirection.StartToEnd -> startIconColor
        DismissDirection.EndToStart -> endIconColor
    }

    val alignment = when (direction) {
        DismissDirection.StartToEnd -> Alignment.CenterStart
        DismissDirection.EndToStart -> Alignment.CenterEnd
    }

    val icon = when (direction) {
        DismissDirection.StartToEnd -> startIcon.icon
        DismissDirection.EndToStart -> endIcon.icon
    }
    val scale by animateFloatAsState(
        if (dismissState.targetValue == DismissValue.Default) 1f else scaleOnDismiss,
        animationSpec = spring(dampingRatio = Spring.DampingRatioHighBouncy)
    )

    Box(
        modifier
            .fillMaxSize()
            .background(backgroundColor)
            .padding(horizontal = 20.dp),
        contentAlignment = alignment
    ) {
        Icon(
            icon,
            contentDescription = "",
            modifier = Modifier.scale(scale),
            tint = iconColor
        )
    }
}

```
```kotlin
data class SwipeableItemBackgrounds(
    val defaultColor: Color,
    val dismissedToEndColor: Color,
    val dismissedToStartColor: Color,
)

data class SwipeableItemIcon(
    val defaultColor: Color,
    val dismissedColor: Color,
    val icon: ImageVector
)
```

Usage:
```kotlin
EasySwipeableBackground(
    dismissState = state,
    backgrounds = SwipeableItemBackgrounds(MaterialTheme.colorScheme.surface, Color.Yellow, Color.Red),
    startIcon = SwipeableItemIcon(Color.Yellow, Color.Black, Icons.Default.Star),
    endIcon = SwipeableItemIcon(Color.Red, Color.Black, Icons.Default.Delete)
)
```
