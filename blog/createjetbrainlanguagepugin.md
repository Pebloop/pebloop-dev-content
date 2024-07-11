# How to create your own language plugin for Jetbrains IDEs

To set the context, I'm working on a flutter project that use blockly. The thing is that blockly is a javascript library. First thing I did was to search a lib that could help me to use blockly in flutter. I found a package called [flutter_blockly](https://pub.dev/packages/flutter_blockly) that is a wrapper for blockly.

The problem is that this package lack a lot of features that I need. So I decided to fork it and create my own package : [Flutter Blockly Plus](https://pub.dev/packages/flutter_blockly_plus) (FBP).

Now when you want to inject javascript into the webview that contain blockly you have to do it either by using a file or by using a string directly in the dart code. In both cases you will lack auto-completion and other usefull developement features and it's unconvenient.

All that lead to the current day where I decided to create a plugin for Jetbrains IDEs that will help me to write the javascript code that I will inject into the webview.

## Step 1: Install the necessary tools

To create a plugin for Jetbrains IDEs you will need **Intellij IDEA**. You can download it from [here](https://www.jetbrains.com/idea/download/).

You'll also need to install 2 plugins, the first is the [gradle plugin](https://plugins.jetbrains.com/plugin/13112-gradle) and the second is the [Plugin devkit](https://plugins.jetbrains.com/plugin/22851-plugin-devkit).

## Step 2: Create a new project

Open Intellij IDEA and create a new project. Choose **IDE Plugin**, fill in the necessary information and click on **Create**.

## Step 3: Create a language

Now that we're set up, we can start creating our language.

Create a new class in the `src/main/kotlin/*your package path*/` directory and name it `*Your language Name*Language.kt`. For example I named it `FBPLanguage.kt`.

The class must inherite `Language`, contain a static (or compagion object) instance of itself and it's contructor must call the super constructor with the name of the language.

```kt
import com.intellij.lang.Language

class FBPLanguage() : Language("FlutterBlocklyPlus") {

    companion object {
        val INSTANCE: FBPLanguage = FBPLanguage();
    }

}
```

## Step 4: Create a file type

Jetbrain IDEs detect files languages by their extension. So we need to create a file type for our language.
Create a ne class in the same directory as before and name it `*Your language Name*FileType.kt`. For example I named it `FBPFileType.kt`.

The class must inherite `LanguageFileType`, contain a static (or compagion object) instance of itself and it's contructor must call the super constructor with the instance of the language we created before.

```kt
class FBPFileType() : LanguageFileType(FBPLanguage.INSTANCE) {

    companion object {
        val INSTANCE: FBPFileType = FBPFileType()
    }
}
```

Now, we need to implement some methods that are required by the `LanguageFileType` class.

```kt
    override fun getName(): String {
        return "FlutterBlocklyPlus"
    }

    override fun getDescription(): String {
        return "FlutterBlocklyPlus file"
    }

    override fun getDefaultExtension(): String {
        return "fbp"
    }

    override fun getIcon(): Icon {
        return
    }
}
```

The methods are kinda self-explanatory :
- `getName` returns the name of the file type.
- `getDescription` returns the description of the file type.
- `getDefaultExtension` returns the default extension of the file type.
- `getIcon` returns the icon of the file type.

For this last method, let's create an icon.

Create a class that will store our icons and store you icon in `src/main/ressources/` directory. You can also create an `icons` directory to sort your resources.

```kt
class FBPIcons {
    companion object {

        val FILE: Icon = getIcon("/icons/pluginIcon.svg", FBPIcons::class.java);
    }
}
```

Now we can add the icon to our file type.

```kt
class FBPFileType() : LanguageFileType(FBPLanguage.INSTANCE) {

    companion object {
        val INSTANCE: FBPFileType = FBPFileType()
    }

    @NotNull
    override fun getName(): String {
        return "FlutterBlocklyPlus"
    }

    override fun getDescription(): String {
        return "FlutterBlocklyPlus file"
    }

    override fun getDefaultExtension(): String {
        return "fbp"
    }

    override fun getIcon(): Icon {
        return FBPIcons.FILE
    }
}
```

## Step 5: Time to test !

Now it's time to ensure that everything is working fine.
In the far right click on the `Gradle` tab and in `intellij` folder execute `runIde`. A new IDE will open, create a new project and create a new file with the extension you defined in the file type. You should see the icon you defined in the file type.

![FlutterBlocklyPlus file](./images/createjetbrainlanguagepugin_icon_in_ide.png)

## Step 6: Code analysis

Now that we have our language set up, we need to analyse the code. It's a neccessity to have auto-completion and other features that will help us to write our "special javascript" code.

But how can we do that ? Well, let's do some theory first.

To analyse code Jetbrain IDEs use a tool called PSI (Program Structure Interface). It's a layer of the IDE that parse the code and create a tree of the code structure. This tree is called AST (Abstract Syntax Tree). Here the analysis process :

![Code analysis](./images/createjetbrainlanguagepugin_code_analysis.png)

As you can see, the code go through a lexer that split the code into tokens. Then the parser will create the AST.
To better understand how it works, let's take an example. Let's say we have the following code :

```js
a = 1 + 2
```

Firstly, our code will go through something called lexer. The lexer will split the code into tokens. In our case it will split the code into elements and convert them into tokens. :

```js
a, =, 1, +, 2 // THe lexer isolate each elements

// The lexer convert the elements into tokens
IDENTIFIER(a), ASSIGN(=), NUMBER(1), PLUS(+), NUMBER(2)
```

![Lexer](./images/createjetbrainlanguagepugin_lexer.png)

After the lexer, the parser will create the AST. The AST is a tree that represent the structure of the code. In our case it will look like this :

```
            ASSIGN(a)
            /       \
IDENTIFIER(a)       PLUS
                    /   \
                NUMBER(1) NUMBER(2)
```
    
![AST](./images/createjetbrainlanguagepugin_parser.png)

Here's the whole process

![final process](./images/createjetbrainlanguagepugin_final_process.png)

Once you have the AST you can analyse the code and do whatever you want with it, like auto-completion, error detection, etc.

Now that we know how it works, let's implement it.