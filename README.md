# Tesseract OCR for PHP

A wrapper to work with Tesseract OCR inside PHP.

## Installation

Via [Composer][]:

    $ composer require IkerST/tesseract_ocr

:bangbang: **This library depends on [Tesseract OCR][], version _3.02_ or later.**



### Note for Windows users

There are [many ways][tesseract_installation_on_windows] to install
[Tesseract OCR][] on your system, but if you just want something quick to
get up and running, I recommend installing the [Capture2Text][] package with
[Chocolatey][].

    choco install capture2text --version 3.9

:warning: Recent versions of Capture2Text stopped shipping the `tesseract` binary.



### Note for macOS users

With [MacPorts][] you can install support for individual languages, like so:
```bash
	sudo port install tesseract-<langcode>
```

But that is not possible with [Homebrew][]. It comes only with **English** support
by default, so if you intend to use it for other language, the quickest solution
is to install them all:

```bash
brew install tesseract --with-all-languages
```



## Usage

### Basic usage

```php
use IkerST\TesseractOCR\TesseractOCR;
echo (new TesseractOCR('text.png'))
    ->run();
```



### Other languages

```php
use IkerST\TesseractOCR\TesseractOCR;
echo (new TesseractOCR('german.png'))
    ->lang('deu')
    ->run();
```



### Multiple languages

```php
use IkerST\TesseractOCR\TesseractOCR;
echo (new TesseractOCR('mixed-languages.png'))
    ->lang('eng', 'jpn', 'spa')
    ->run();
```



### Inducing recognition

```php
use IkerST\TesseractOCR\TesseractOCR;
echo (new TesseractOCR('8055.png'))
    ->whitelist(range('A', 'Z'))
    ->run();
```



## API

### image

Define the path of an image to be recognized by `tesseract`.

```php
$ocr = new TesseractOCR();
$ocr->image('/path/to/image.png');
$ocr->run();
```

### imageData

Set the image to be recognized by `tesseract` from a string, with its size.
This can be useful when dealing with files that are already loaded in memory.
You can easily retrieve the image data and size of an image object :
```php
//Using Imagick
$data = $img->getImageBlob();
$size = $img->getImageLength();
//Using GD
ob_start();
// Note that you can use any format supported by tesseract
imagepng($img, null, 0);
$size = ob_get_length();
$data = ob_get_clean();

$ocr = new TesseractOCR();
$ocr->imageData($data, $size);
$ocr->run();
```

### executable

Define a custom location of the `tesseract` executable,
if by any reason it is not present in the `$PATH`.

```php
echo (new TesseractOCR('img.png'))
    ->executable('/path/to/tesseract')
    ->run();
```

### version

Returns the current version of `tesseract`.

```php
echo (new TesseractOCR())->version();
```

### availableLanguages

Returns a list of available languages/scripts.

```php
foreach((new TesseractOCR())->availableLanguages() as $lang) echo $lang;
```

__More info:__ <https://github.com/tesseract-ocr/tesseract/blob/master/doc/tesseract.1.asc#languages-and-scripts>

### tessdataDir

Specify a custom location for the tessdata directory.

```php
echo (new TesseractOCR('img.png'))
    ->tessdataDir('/path')
    ->run();
```

### userWords

Specify the location of user words file.

This is a plain text file containing a list of words that you want to be
considered as a normal dictionary words by `tesseract`.

Useful when dealing with contents that contain technical terminology, jargon,
etc.

```
$ cat /path/to/user-words.txt
foo
bar
```

```php
echo (new TesseractOCR('img.png'))
    ->userWords('/path/to/user-words.txt')
    ->run();
```

### userPatterns

Specify the location of user patterns file.

If the contents you are dealing with have known patterns, this option can help
a lot tesseract's recognition accuracy.

```
$ cat /path/to/user-patterns.txt'
1-\d\d\d-GOOG-441
www.\n\\\*.com
```

```php
echo (new TesseractOCR('img.png'))
    ->userPatterns('/path/to/user-patterns.txt')
    ->run();
```

### lang

Define one or more languages to be used during the recognition.
A complete list of available languages can be found at:
<https://github.com/tesseract-ocr/tesseract/blob/master/doc/tesseract.1.asc#languages>

Use the combination `->lang('chi_sim', 'chi_tra')`
for proper recognition of Chinese.

```php
 echo (new TesseractOCR('img.png'))
     ->lang('lang1', 'lang2', 'lang3')
     ->run();
```

### psm

Specify the Page Segmentation Method, which instructs `tesseract` how to
interpret the given image.

__More info:__ <https://github.com/tesseract-ocr/tesseract/wiki/ImproveQuality#page-segmentation-method>

```php
echo (new TesseractOCR('img.png'))
    ->psm(6)
    ->run();
```

### oem

Specify the OCR Engine Mode. (see `tesseract --help-oem`)

```php
echo (new TesseractOCR('img.png'))
    ->oem(2)
    ->run();
```

### whitelist

This is a shortcut for `->config('tessedit_char_whitelist', 'abcdef....')`.

```php
echo (new TesseractOCR('img.png'))
    ->whitelist(range('a', 'z'), range(0, 9), '-_@')
    ->run();
```

### configFile

Specify a config file to be used. It can either be the path to your own
config file or the name of one of the predefined config files:
<https://github.com/tesseract-ocr/tesseract/tree/master/tessdata/configs>

```php
echo (new TesseractOCR('img.png'))
    ->configFile('hocr')
    ->run();
```

### setOutputFile

Specify an Outputfile to be used. Be aware: If you set an outputfile then
the option `withoutTempFiles` is ignored.
Tempfiles are written (and deleted) even if `withoutTempFiles = true`.

In combination with `configFile` you are able to get the `hocr`, `tsv` or
`pdf` files.

```php
echo (new TesseractOCR('img.png'))
    ->configFile('pdf')
    ->setOutputFile('/PATH_TO_MY_OUTPUTFILE/searchable.pdf');
    ->run();
```

### digits

Shortcut for `->configFile('digits')`.

```php
echo (new TesseractOCR('img.png'))
    ->digits()
    ->run();
```

### hocr

Shortcut for `->configFile('hocr')`.

```php
echo (new TesseractOCR('img.png'))
    ->hocr()
    ->run();
```

### pdf

Shortcut for `->configFile('pdf')`.

```php
echo (new TesseractOCR('img.png'))
    ->pdf()
    ->run();
```

### quiet

Shortcut for `->configFile('quiet')`.

```php
echo (new TesseractOCR('img.png'))
    ->quiet()
    ->run();
```

### tsv

Shortcut for `->configFile('tsv')`.

```php
echo (new TesseractOCR('img.png'))
    ->tsv()
    ->run();
```

### txt

Shortcut for `->configFile('txt')`.

```php
echo (new TesseractOCR('img.png'))
    ->txt()
    ->run();
```

### tempDir

Define a custom directory to store temporary files generated by tesseract.
Make sure the directory actually exists and the user running `php` is allowed
to write in there.

```php
echo (new TesseractOCR('img.png'))
    ->tempDir('./my/custom/temp/dir')
    ->run();
```

### withoutTempFiles

Specify that `tesseract` should output the recognized text without writing to temporary files.
The data is gathered from the standard output of `tesseract` instead.

```php
echo (new TesseractOCR('img.png'))
    ->withoutTempFiles()
    ->run();
```

### Other options

Any configuration option offered by Tesseract can be used like that:

```php
echo (new TesseractOCR('img.png'))
    ->config('config_var', 'value')
    ->config('other_config_var', 'other value')
    ->run();
```

Or like that:

```php
echo (new TesseractOCR('img.png'))
    ->configVar('value')
    ->otherConfigVar('other value')
    ->run();
```

__More info:__ <https://github.com/tesseract-ocr/tesseract/wiki/ControlParams>

### Thread-limit

Sometimes, it may be useful to limit the number of threads that tesseract is
allowed to use (e.g. in [this case](https://github.com/tesseract-ocr/tesseract/issues/898)).
Set the maxmium number of threads as param for the `run` function:

```php
echo (new TesseractOCR('img.png'))
    ->threadLimit(1)
    ->run();
```