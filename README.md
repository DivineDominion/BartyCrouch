<p align="center">
    <img src="https://raw.githubusercontent.com/Flinesoft/BartyCrouch/develop/Logo.png"
      width=600 height=167>
</p>

<p align="center">
    <a href="https://github.com/Flinesoft/BartyCrouch/tags">
        <img src="https://img.shields.io/github/tag/Flinesoft/BartyCrouch.svg"
             alt="GitHub tag">
    </a>
    <a href="#">
        <img src="https://img.shields.io/badge/Swift-2.1-DD563C.svg"
             alt="Swift version">
    </a>
    <a href="https://github.com/Flinesoft/BartyCrouch/blob/develop/LICENSE.md">
        <img src="https://img.shields.io/github/license/Flinesoft/BartyCrouch.svg"
              alt="GitHub license">
    </a>
</p>


# BartyCrouch

BartyCrouch can **search a Storyboard/xib file for localizable strings** and **update your existing localization `.strings` incrementally** by adding new keys, keeping your existing translations and deleting only the ones that are no longer used. BartyCrouch even **keeps changes to your translation comments** given they are enclosed like `/* comment to keep */` and don't span multiple lines.

Additionally BartyCrouch can now also **automatically translate existing string files to languages you don't speak** using the Microsoft Translator API. You can exactly **choose the languages to auto-translate** and BartyCrouch will **keep all existing translations** by default.


## Installation

Install Homebrew first if you don't have it already (more about Homebrew [here](http://brew.sh)):
``` shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Then simply run the commands
``` shell
brew tap flinesoft/bartycrouch
brew install flinesoft/bartycrouch/bartycrouch
```
to install BartyCrouch.


## Usage

Before using BartyCrouch please **make sure you have committed your code**.

### Complete Examples (TL;DR)

With BartyCrouch you can run commands like these:

``` shell
# Incrementally update English strings of Main.storyboard
bartycrouch -i "path/Base.lproj/Main.storyboard" -o "path/en.lproj/Main.strings"

# Incrementally update English and German strings of Main.storyboard
bartycrouch -i "path/Base.lproj/Main.storyboard" -o "path/en.lproj/Main.strings" "path/de.lproj/Main.strings"

# Incrementally update all languages of Main.storyboard
bartycrouch -i "path/Base.lproj/Main.storyboard" -a
```

Also you can make your life a lot easier by using the **build script method** described [below](#build-script).

### Commands Overview

The `bartycrouch` main command accepts one of the following combinations of arguments:

1. Input and Output/Auto/Except
2. Translate with Input and Output/Auto/Except

You can also additionally specify Force and/or Verbose on each command.

#### Input (aka `-i`)

You can specify the input Storyboard or XIB file using `-i "path/to/my.storyboard"` (`-i` is short `--input`).

#### Output (aka `-o`)

You can pass a list of `.strings` files to be incrementally updated / translated using  `-o "path/to/en.strings" "path/to/de.strings"` (`-o` is short for `--output`).

#### Auto (aka `-a`)

If you use base internationalization (recommended) you can also let BartyCrouch find and update all `.strings` files automatically by passing `--auto` or `-a` using the shorthand syntax.

#### Except (aka `-e`)

Sometimes you may be supporting a bunch of languages and want to translate to all of them except for one or two you want to deal with differently. In these cases you can specify a list of paths to exclude with `-e "path/to/your.file"` (`-e` is short for `--except`).

#### Translate (aka `-t`)

Sometimes it makes sense to start with machine translated strings and let humans improve them later on. This can save time and even be a better solution for some languages you might not have localized your app to otherwise.

You can now do this easily with BartyCrouch: Simply run the bartycrouch command with a `.strings` file as input instead of a Storyboard/XIB file and add `-t "{ id: YOUR_ID }|{ secret: YOUR_SECRET }"` (`-t` is short for `--translate`). A simple example:

``` shell
bartycrouch -t "{ id: MyApp }|{ secret: abc123 }" -i "path/to/en.lproj/Localizable.strings" -a
# => translates all empty values for all languages (except english) using the english translations as input

bartycrouch -t "{ id: MyApp }|{ secret: abc123 }" -i "path/to/en.lproj/Localizable.strings" -e "path/to/de.lproj/Localizable.strings" -f
# => force-translates all values – also existing ones - for all languages except German (and English)
```

In order to use the Microsoft Translator API you need to register [here](https://datamarket.azure.com/dataset/bing/microsofttranslator) (there's a very useful free option). Then you can add a client [here](https://datamarket.azure.com/developer/applications) which will give you the credentials needed for this feature.

#### Force (aka `-f`)

In case you don't want to keep existing translations but want BartyCrouch to overwrite all of existing translations that you can pass the `-f` option (`-f` is short for `--force`).

#### Verbose (aka `-v`)

In case commands manually typed in the command line take too long or your simply want to know more about what is happening you can also run all commands with the `-v` flag (`-v` is short for `--verbose`). This will print more details about the current work in progress.



### Build Script

You may want to **update your `.strings` files on each build automatically** what you can easily do by adding a run script to your target in Xcode. In order to do this select your target in Xcode, choose the `Build Phases` tab and click the + button on the top left corner of that pane. Select `New Run Script Phase` and copy the following into the text box below the `Shell: /bin/sh` of your new run script phase:

``` shell
if which bartycrouch > /dev/null; then
    # Set path to base internationalized Storyboard/XIB files
    BASE_PATH="$PROJECT_DIR/Sources/Base.lproj"

    # Incrementally update all Storyboards/XIBs strings files
    bartycrouch -i "$BASE_PATH/Main.storyboard" -a
    bartycrouch -i "$BASE_PATH/LaunchScreen.storyboard" -a
    bartycrouch -i "$BASE_PATH/CustomView.xib" -a

    # Set Microsoft Translator API credentials
    EN_PATH="$PROJECT_DIR/Sources/en.lproj"
    CREDS="{ id: YOUR_ID }|{ secret: YOUR_SECRET }"

    # Machine-translate empty language values for all languages
    bartycrouch -t $CREDS -i "$EN_PATH/Localizable.strings" -a
    bartycrouch -t $CREDS -i "$EN_PATH/Main.strings" -a
    bartycrouch -t $CREDS -i "$EN_PATH/LaunchScreen.strings" -a
else
    echo "BartyCrouch not installed, download it from https://github.com/Flinesoft/BartyCrouch"
fi
```

<img src="Build-Script-Example.png">

Now update the `BASE_PATH` to point to your Base.lproj directory, add a `bartycrouch -i ... -a` for each of your base internationalized Storyboards/XIBs (if any) and you're good to go. You should also uncomment or remove the lines below `# Set Microsoft ...` until `bartycrouch -t ...` if you don't want to use the machine translation feature. Xcode will now run BartyCrouch each time you build your project and update your `.strings` files accordingly.

*Note: Please make sure you commit your code using source control regularly when using the build script method.*

### Exclude specific views from localization

Sometimes you may want to **ignore some specific views** containing localizable texts e.g. because **their values are set programmatically**.
For these cases you can simply include `#bartycrouch-ignore!` or the shorthand `#bc-ignore!` into your value within your base localized Storyboard/XIB file.
This will tell BartyCrouch to ignore this specific view when updating your `.strings` files.

Here's an example of how a base localized view in a XIB file with partly ignored strings might look like:

<img src="Exclusion-Example.png">

## Migration Guides

This project follows [Semantic Versioning](http://semver.org). Please follow the appropriate guide below when **upgrading to a new major version** of BartyCrouch (e.g. 0.3 -> 1.0).

### Upgrade from 0.x to 1.x

- `--input-storyboard` and `-in` were **renamed** to `--input` and `-i`
- `--output-strings-files` and `-out` were **renamed** to `--output` and `-o`
- Multiple paths passed to `-output` are now **separated by whitespace instead of comma**
  - e.g. `-out "path/one,path/two"` should now be `-o "path/one" "path/two"`
- `--output-all-languages` and `-all` were **renamed** to `--auto` and `-a`


## Contributing

Contributions are welcome. Please just open an Issue on GitHub to discuss a point or request a feature there or send a Pull Request with your suggestion. Please also make sure to write tests for your changes in order to make sure they don't break in the future. Please note that there is a framework target within the project alongside the command line utility target to make testing easier.


## License
This library is released under the [MIT License](http://opensource.org/licenses/MIT). See LICENSE for details.