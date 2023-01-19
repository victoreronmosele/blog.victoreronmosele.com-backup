# Rebuilding The Flutter Counter App In Jetpack Compose

This article will show how to rebuild the default [Flutter](https://flutter.dev/) counter app using [Jetpack Compose](https://developer.android.com/jetpack/compose).

The article is divided into two sections:

1. [UI](#ui), where the app's user interface will be built.
    
2. [State](#state), where the state of the app's counter will be implemented.
    

Below is a screenshot of the default Flutter counter app:

![162827266517692828 (3).jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1628273185586/9AM_bWM-k.jpeg align="left")

This is what we'll rebuild with Jetpack Compose.

The first step is to download a [Jetpack Compose supported Android Studio](https://developer.android.com/studio) version (at this time of writing, that is the Arctic Fox version).

The next step is to create an empty compose activity by clicking on File &gt; New Project and selecting `Empty Compose Activity` from the templates shown.

Click on `Next`.

Then we enter our application name. For this article, we'll use "ComposeCounterApp".

Click on `Finish` and it should open the `ComposeCounterApp` project.

## 1\. <span id="ui">UI </span>

Open the `MainActivity.kt` file. It should contain the default Compose Activity code.

We'll remove the code and update the content of `MainActivity.kt` to this below:

```kotlin
package com.example.composecounterapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.material.Scaffold
import androidx.compose.runtime.Composable
import com.example.composecounterapp.ui.theme.ComposeCounterAppTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ComposeCounterAppTheme {
                ComposeCounter()
            }
        }
    }
}

@Composable
fun ComposeCounter() {
    Scaffold(
        content = {}
    )
}
```

On running the project on a device, an empty screen is displayed. This is because `ComposeCounter` only contains a [Scaffold](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#Scaffold) composable with empty content.

Next, we'll add the code for the app bar. We'll use the [TopAppBar](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#TopAppBar) composable and specify the title, the background colour, and the content colour.

Update the `ComposeCounter` function to this.

```kotlin
@Composable
fun ComposeCounter() {
    val primaryColor = Color(0xFF2196F3)

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(text = "Compose Home Demo Page") },
                backgroundColor = primaryColor,
                contentColor = Color.White
            )
        },
        content = {}
    )
}
```

This will show the following screen:

![162827266517692828.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1628272825854/iNS_NCNHS.jpeg align="left")

Next, we'll add the content of the screen. And for this, we'll pass a Column with two Texts to the Scaffold as `content`. The column items will be aligned in the center and we'll pass [modifiers](https://developer.android.com/reference/kotlin/androidx/compose/ui/Modifier) to ensure the Column fills the screen horizontally and vertically.

```kotlin
@Composable
fun ComposeCounter() {
    val primaryColor = Color(0xFF2196F3)

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(text = "Compose Home Demo Page") },
                backgroundColor = primaryColor,
                contentColor = Color.White
            )
        },
        content = {
            Column(
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally,
                modifier = Modifier
                    .fillMaxHeight()
                    .fillMaxWidth()
            ) {
                Text(
                    text = "You have pushed this button this many times",
                )
                Text(
                    text = "0",
                    style = MaterialTheme.typography.h4,
                )
            }
        },
    )
}
```

And now we'll have this showing on our device.

![162827266517692828 (1).jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1628272968634/ZF1NNk3dE.jpeg align="left")

The last thing to add for the UI is the floating action button, which we can specify on the Scaffold by passing a [FloatingActionButton](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary#FloatingActionButton) composable to the Scaffold as `floatingActionButton`.

We do that by updating the `ComposeCounter` composable to the code below:

```kotlin
@Composable
fun ComposeCounter() {
    val primaryColor = Color(0xFF2196F3)

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(text = "Compose Home Demo Page") },
                backgroundColor = primaryColor,
                contentColor = Color.White
            )
        },
        content = {
            Column(
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally,
                modifier = Modifier
                    .fillMaxHeight()
                    .fillMaxWidth()
            ) {
                Text(
                    text = "You have pushed this button this many times",
                )
                Text(
                    text = "0",
                    style = MaterialTheme.typography.h4,
                )
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {},
                backgroundColor = primaryColor,
                contentColor = Color.White
            ) {
                Icon(Icons.Filled.Add, contentDescription = "Add button")
            }
        },
    )
}
```

And now, we'll have our complete UI! 🎉

![162827266517692828 (2).jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1628273060642/L88MXLIwy.jpeg align="left")

## 2\. State

Currently, the app displays "0" as the counter state and we need to make sure that clicking the floating action button increments that state.

We need a variable to hold the state of the counter.

From [State and Jetpack Compose](https://developer.android.com/jetpack/compose/state),

> There are three ways to declare a MutableState object in a composable:

> * val mutableState = remember { mutableStateOf(default) }
>     
> * var value by remember { mutableStateOf(default) }
>     
> * val (value, setValue) = remember { mutableStateOf(default) }
>     

We'll be using the second one.

At the top of the `ComposeCounter` composable, we'll add a variable `counter` to hold the state of the counter and set its default value to zero.

We'll also use this `counter` variable in the Text composable instead of "0".

And now the `ComposeCounter` will have the following code:

```kotlin
@Composable
fun ComposeCounter() {
    val primaryColor = Color(0xFF2196F3)
    var counter: Int by remember { mutableStateOf(0) }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(text = "Compose Home Demo Page") },
                backgroundColor = primaryColor,
                contentColor = Color.White
            )
        },
        content = {
            Column(
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally,
                modifier = Modifier
                    .fillMaxHeight()
                    .fillMaxWidth()
            ) {
                Text(
                    text = "You have pushed this button this many times",
                )
                Text(
                    text = counter.toString(),
                    style = MaterialTheme.typography.h4,
                )
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {
                },
                backgroundColor = primaryColor,
                contentColor = Color.White
            ) {
                Icon(Icons.Filled.Add, contentDescription = "Add button")
            }
        },
    )
}
```

There will be no visible change on the screen for now.

The next thing to do is to increment the counter when the floating action button is clicked.

We can do that in the `onClick` callback of the `FloatingActionButton`.

The code will be updated to this:

```kotlin
@Composable
fun ComposeCounter() {
    val primaryColor = Color(0xFF2196F3)
    var counter: Int by remember { mutableStateOf(0) }


    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(text = "Compose Home Demo Page") },
                backgroundColor = primaryColor,
                contentColor = Color.White
            )
        },
        content = {
            Column(
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally,
                modifier = Modifier
                    .fillMaxHeight()
                    .fillMaxWidth()
            ) {
                Text(
                    text = "You have pushed this button this many times",
                )
                Text(
                    text = counter.toString(),
                    style = MaterialTheme.typography.h4,
                )
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {
                    counter += 1
                },
                backgroundColor = primaryColor,
                contentColor = Color.White
            ) {
                Icon(Icons.Filled.Add, contentDescription = "Add button")
            }
        },
    )
}
```

Now the app increments the counter when the floating action button is tapped as seen in the gif below:

![162827266517692828.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1628273301925/_R-MgXVVO.gif align="left")

And now the app is complete.

The complete code for the `MainActivity.kt` is this below:

```kotlin
package com.example.composecounterapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxHeight
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.material.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import com.example.composecounterapp.ui.theme.ComposeCounterAppTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ComposeCounterAppTheme {
                ComposeCounter()
            }
        }
    }
}

@Composable
fun ComposeCounter() {
    val primaryColor = Color(0xFF2196F3)
    var counter: Int by remember { mutableStateOf(0) }


    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text(text = "Compose Home Demo Page") },
                backgroundColor = primaryColor,
                contentColor = Color.White
            )
        },
        content = {
            Column(
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally,
                modifier = Modifier
                    .fillMaxHeight()
                    .fillMaxWidth()
            ) {
                Text(
                    text = "You have pushed this button this many times",
                )
                Text(
                    text = counter.toString(),
                    style = MaterialTheme.typography.h4,
                )
            }
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = {
                    counter += 1
                },
                backgroundColor = primaryColor,
                contentColor = Color.White
            ) {
                Icon(Icons.Filled.Add, contentDescription = "Add button")
            }
        },
    )
}
```

The complete project can be found on [Github](https://github.com/victoreronmosele/compose-counter-app).