Article: [Introduction to Jetpack DataStore | by Simona Stojanovic | Android Developers | Medium](https://medium.com/androiddevelopers/introduction-to-jetpack-datastore-3dc8d74139e7)

Codelab: [Working with Proto DataStore](https://developer.android.com/codelabs/android-proto-datastore#0)

## **Protocol Buffers:**

Protocol Buffers is a way to store data in a way that is easy to read and write. It is **language-neutral**, which means that it can be used in any programming language. It is also **platform-neutral**, which means that it can be used on any platform. And it is extensible, which means that you can add new data types to it without breaking existing data.

Here is an example of how Protocol Buffers can be used. Suppose you have a program that stores information about people. You could use Protocol Buffers to store the person's name, age, and address. The data would be stored in a **binary format**, which makes it smaller and faster than storing it in text format. And because Protocol Buffers is language-neutral, you could use the same data in a program written in Java, C++, or any other programming language.

via: [Bard](https://bard.google.com)

## **Working with Proto Datastore**

Proto Datastore uses [Protocol buffers](https://www.notion.so/Jetpack-datastore-f1241300b4ad4da1a3ebc4cc7b978429?pvs=21) to define the schema. Using Protos Datastore knows what types are stored and will just provide them, removing the need for using keys.

### ****Adding dependencies:****

app level:
```groovy
plugins {
    ...
    id "com.google.protobuf" version "0.9.1"
}

dependencies {
    implementation  "androidx.datastore:datastore:1.0.0"
    implementation  "com.google.protobuf:protobuf-javalite:3.18.0"
    ...
}

protobuf {
    // Configures the Protobuf compilation and the protoc executable
    protoc {
        // Downloads from the repositories
        artifact = "com.google.protobuf:protoc:3.14.0"
    }

    // Generates the java Protobuf-lite code for the Protobufs in this project
    generateProtoTasks {
        all().each { task ->
            task.builtins {
            // Configures the task output type
                java {
            // Java Lite has smaller code size and is recommended for Android
                    option 'lite'
                }
            }
        }
    }
 }
```

*💡 Quick tip — if you want to minify your build, make sure to add an additional rule to your `proguard-rules.pro` file to prevent your fields from being deleted:*

```kotlin
-keepclassmembers class * extends androidx.datastore.preferences.protobuf.GeneratedMessageLite {
    <fields>;
}
```

### ****Defining and using protobuf objects:****

Protocol buffers are a mechanism for serializing structured data. You define how you want your data to be structured once and then the compiler generates source code to easily write and read the structured data.

*app/src/main/proto*

`UserPreferences` — `.proto` schema:

```protobuf
syntax = "proto3";

option java_package = "*$**PACKAGE_NAME**$*";
option java_multiple_files = true;

message UserPreferences {

    bool show_completed = 1;
    SortOrder sort_order = 2;

    enum SortOrder {
        UNSPECIFIED = 0;
        NONE = 1;
        BY_DEADLINE = 2;
        BY_PRIORITY = 3;
        BY_DEADLINE_AND_PRIORITY = 4;
    }
}
```

- `syntax` — specifies that you’re using `[proto3](https://developers.google.com/protocol-buffers/docs/proto3)` syntax
- `java_package` — file option that specifies **package declaration** for your generated classes, which helps prevent **naming conflicts** between different projects
- `java_multiple_files` — Use the file option to specify whether a single file or separate files will be generated for each top-level message type in the `.proto`. By default, a single file with nested subclasses is generated.

⚠ *Make sure you rebuild the project.*

*app\src\main\java\…\data\datastore\*

`UserPreferences` — Kotlin data class:

```kotlin
data class UserPreferences(
    val showCompleted: Boolean,
    val sortOrder: SortOrder
)

enum class SortOrder {
    UNSPECIFIED,
    NONE,
    BY_DEADLINE,
    BY_PRIORITY,
    BY_DEADLINE_AND_PRIORITY
}
```

`UserPreferencesSerializer` — kotlin object:

To tell DataStore how to read and write the data type we defined in the proto file, we need to implement a `Serializer`. The Serializer defines also the default value to be returned if there's no data on disk. Create a new file called `UserPreferencesSerializer`:

```kotlin
object UserPreferencesSerializer : Serializer<UserPreferences> {
    override val defaultValue: UserPreferences = UserPreferences.getDefaultInstance()
    override suspend fun readFrom(input: InputStream): UserPreferences {
        try {
            return UserPreferences.parseFrom(input)
        } catch (exception: InvalidProtocolBufferException) {
            throw CorruptionException("Cannot read proto.", exception)
        }
    }

    override suspend fun writeTo(t: UserPreferences, output: OutputStream) = 
				t.writeTo(output)
}
```

*💡 Quick tip — If at any point your AS is unable to find anything `UserPreferences` related, **clean and rebuild** your project to initiate the generation of the protobuf classes.*

### ****Creating a Proto Datastore****

You interact with Proto through an instance of `DataStore<UserPreferences>`. `DataStore` is an interface that **grants access to the persisted information**, in our case in the form of the generated `UserPreferences`.

To create this instance, it is recommended to use the delegate `dataStore` and pass mandatory `fileName` and `serializer` arguments:

```kotlin
private const val DATA_STORE_FILE_NAME = "user_prefs.pb"

private val Context.userPreferencesStore: DataStore<UserPreferences> by dataStore(
    fileName = DATA_STORE_FILE_NAME,
    serializer = UserPreferencesSerializer,
		corruptionHandler = ReplaceFileCorruptionHandler(
    produceNewData = { UserPreferences.getDefaultInstance() }
	)    
)
```

`corruptionHandler` is invoked if a `CorruptionException` is thrown by the serializer when the **data cannot be de-serialized**. `corruptionHandler` would then instruct Proto how to replace the corrupted data.

⚠ *Only create one Datastore instance per file to avoid breaking functionality. Add the delegate construction at the top level of your Kotlin file and use it throughout your application as a singleton.*

### ****Reading data****

```kotlin
val userPreferencesFlow: Flow<UserPreferences> = userPreferencesStore.data
```

*🚨 **Do not cache Proto data**. This would invalidate Datastore's data consistency guarantee. To get a single snapshot of your data without subscribing to further Flow emissions, use `userPreferencesStore.data.first()`.*

```kotlin
// Don't
suspend fun fetchCachedPrefs(scope: CoroutineScope): StateFlow<UserPreferences> =
   userPreferencesStore.data.stateIn(scope)

// Do
suspend fun fetchInitialPreferences() = userPreferencesStore.data.first()
```

### ****Writing data****

```kotlin
suspend fun updateShowCompleted(showCompleted: Boolean) {
      userPreferencesStore.updateData { currentPreferences ->
            currentPreferences.toBuilder().setShowCompleted(completed).build()
        }
}
```

- `toBuilder()` — gets the `Builder` version of our `currentPreferences` which “unlocks” it for changes
- `.setShowCompleted(completed)` — sets the new value
- `.build()` — finishes the update process by converting it back to `UserPreferences`

### ****Exception handling****

```kotlin
val userPreferencesFlow: Flow<UserPreferences> = userPreferencesStore.data
    .catch { exception ->
        if (exception is IOException) {
            Log.e(TAG, "Error reading sort order preferences.", exception)
            emit(UserPreferences.getDefaultInstance())
        } else {
            throw exception
        }
    }
```

```kotlin
suspend fun updateShowCompleted(completed: Boolean) {
    try {
        userPreferencesStore.updateData { currentPreferences ->
            currentPreferences.toBuilder().setShowCompleted(completed).build()
        }
    } catch (e: IOException) {
        // Handle error
    }
}
```

### ****Injecting Preferences Datastore****

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DataStoreModule {

    @Provides
    @Singleton
    fun providesUserPreferencesDataStore(
        @ApplicationContext context: Context,
        @Dispatcher(IO) ioDispatcher: CoroutineDispatcher,
        userPreferencesSerializer: UserPreferencesSerializer,
    ): DataStore<UserPreferences> =
        DataStoreFactory.create(
            serializer = userPreferencesSerializer,
            scope = CoroutineScope(ioDispatcher + SupervisorJob()),
        ) {
            context.dataStoreFile("user_preferences.pb")
        }
}
```

```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val userPreferencesRepository: UserPreferencesRepository
) : ViewModel() { }

class UserPreferencesRepository @Inject constructor(
    private val dataStore: DataStore<Preferences>
)
```
