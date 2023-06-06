# compose-instant
I might forget how to implement this

## Easy Collapsing Toolbar

```
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun Screen() {
    val scrollBehavior = TopAppBarDefaults.enterAlwaysScrollBehavior()

    Scaffold(
        modifier = Modifier
                   .nestedScroll(scrollBehavior.nestedScrollConnection),
        floatingActionButton = {
           // FAB
        },
        topBar = {
            TopAppBar(
                title = {
                    Text(text = "My APP", style = MaterialTheme.typography.titleLarge)
                },
                scrollBehavior = scrollBehavior,
                )
        }
    ) {
       // CONTENT
    }
}
```
