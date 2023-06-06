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
