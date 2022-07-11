# Android_Hilt_MVVM_Retrofit
How to develop Android kotlin project in MVVM by Hilt and Retrofit (+ Picasso, NavHostFragment)

## Configuration and Dependencies
### 1- Add or edit dependencies in build.gradle in Module:

```groovy
...
dependencies {
    def dagger_version = "2.41"
    def lifecycle_version = "2.5.0-alpha06"
    def nav_version = "2.4.2"
    def retrofit_version = "2.9.0"
    def picasso_version = "2.71828"

    //Old Dependencies:
    implementation 'androidx.core:core-ktx:1.8.0'
    implementation 'androidx.appcompat:appcompat:1.4.2'
    implementation 'com.google.android.material:material:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'

    //For NavHostFragment:
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
    implementation "androidx.datastore:datastore:1.0.0"
    implementation "androidx.datastore:datastore-preferences:1.0.0"

    //Retrofit
    implementation "com.squareup.retrofit2:retrofit:$retrofit_version"
    implementation "com.squareup.retrofit2:converter-gson:$retrofit_version"
    implementation 'com.jakewharton.retrofit:retrofit2-kotlin-coroutines-adapter:0.9.2'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.2.1'

    //Pcaso:
    implementation "com.squareup.picasso:picasso:$picasso_version"

    //MVVM:
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    implementation "androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version"

    //Hilt:
    implementation "com.google.dagger:hilt-android:$dagger_version"
    kapt "com.google.dagger:hilt-compiler:$dagger_version"
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.0'
}
```
*Note: Dont sync gradle after each step! I will tell you in one step to sync :)*

### 2- Edit settings.gradle:
```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }

    resolutionStrategy {
        eachPlugin {
            if (requested.id.id == 'dagger.hilt.android.plugin') {
                useModule("com.google.dagger:hilt-android-gradle-plugin:2.38.1")
            }
        }
    }
}
...
```

### 3- Edit build.gradle and add this id's:
```groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
    id 'dagger.hilt.android.plugin'
    id 'kotlin-parcelize'
}
```

### 4- Sync gradle and if you see error in invalid kapt in dependencies, comment kapt line, sync once and after SUCCESSFULLY BUILD, uncomment kapt line and sync again to see SUCCESSFULLY BUILD again.


## Hilt and MVVM Annotations
### 1- Add Application class and add @HiltAndroidApp to this class:
```kotlin
package *.*.*

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class App : Application() {
    override fun onCreate() {
        super.onCreate()
    }
}
```
*Note: Add app:name to Manifest.xml file to work this class*

### 2- Add AppModule class and add this annotations to this class and its methods like this (I used Retrofit to get data):
```kotlin
package *.*.*.di

import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import javax.inject.Singleton

@Module
@InstallIn(SingletonComponent::class)
class AppModule {

    @Provides
    fun retrofit() : Retrofit = Retrofit.Builder()
        .baseUrl("https://api.*.*/v1/")
        .client(OkHttpClient.Builder().addInterceptor(HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY
        }).build())
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    @Provides
    @Singleton
    fun apiService(retrofit: Retrofit) : ApiService = retrofit.create(ApiService :: class.java)

    @Provides
    fun repository(apiService: ApiService) : Repository = Repository(apiService)
}
```

### 3- Add ApiService class:
```kotlin
package *.*.*.data

import *.*.*.data.model.Result
import retrofit2.Response
import retrofit2.http.GET

interface ApiService {

    @GET("users?skip=0&limit=100")
    suspend fun items() : Response<Result>
}
```

### 4- Add Repository class:
```kotlin
package *.*.*.data.repository

import *.*.*.data.ApiService

class Repository constructor(private val apiService: ApiService) {
    suspend fun getItems() = apiService.items()
}
```

### 5- Add model class:
```kotlin
package *.*.*.data.model

import com.google.gson.annotations.SerializedName

data class Result(@SerializedName("users") val data: List<User> = listOf())
```

### 6- Add @AndroidEntryPoint to Activity and Fragment class.

### 7- Add ViewModel class like this:
```kotlin
package *.*.*.ui.viewModels

import android.util.Log
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import dagger.hilt.android.lifecycle.HiltViewModel
import *.*.*.data.model.User
import *.*.*.data.repository.Repository
import kotlinx.coroutines.launch
import java.lang.Exception
import javax.inject.Inject

@HiltViewModel
class ItemViewModel @Inject constructor(private val repository: Repository) : ViewModel() {

    private val _itemList = MutableLiveData<List<User>>()
    val itemList get() = _itemList

    fun fetchItems() {
        viewModelScope.launch {
            try {
                val response = repository.getItems()
                with(response) {
                    if(isSuccessful) {
                        body()?.let {
                            _itemList.postValue(it.data)
                        }
                    } else {
                        Log.d("TAG", "fetch items: ${message()}")
                    }
                }
            } catch (e: Exception) {
                Log.d("TAG", "fetch items 2: ${e.message}")
            }
        }
    }

    fun sort(text:String){
        when(text){
            "ID" ->  {_itemList.postValue(_itemList.value?.sortedWith(compareBy { data-> data.id}))}
            "Role" -> {_itemList.postValue(_itemList.value?.sortedBy { data-> data.role})}
            "Name" -> {_itemList.postValue(_itemList.value?.sortedBy { data-> data.name})}
        }
    }
}
```

If you have any issue with these steps, tell me :)
