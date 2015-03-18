# Extracting PO Files automatically #

Have a look at the file **index.php** and modify it to suit your needs.

# Step 0 #
Open **index.php** and replace line 17 by including your smarty class and delete line 19

The start of the file should look somewhat like this after doing the above (sorry for all the fuzz in lines 10-15. I just didn't want to maintain this package outside the whole framework... It's not as bad as it looks...)

```
<?php
/**
  * THIS FILE IS AN EXAMPLE OF HOW TO CREATE
  * ONE BIG PHP FILE FROM ALL TEMPLATES IN THE 
  * TEMPLATE DIRECTORY
  * THE CREATED PHP FILE CAN BE USED WITH with e.g. xgettext (see comment in step DONE.)
  *
  */
DEFINE('BASE_PATH', __DIR__.DIRECTORY_SEPARATOR);
include BASE_PATH.'../../../mebb_functions_glob_recursive.php';
include BASE_PATH.'../../../mebb_i18n_smarty.php';

if(file_exists(BASE_PATH.'../../../../smarty/Smarty.class.php')){
  include_once BASE_PATH.'../../../../smarty/Smarty.class.php';
  include_once BASE_PATH.'../../../../../app/core/web/smarty/functions/locale.php';
}else{
  include '/path/to/my/Smarty.class.php';
  include BASE_PATH.'../../../mebb_i18n_smarty_function_locale.php';
}
```

# Step 1 #
Load smarty as you usually do alongside with all your plugins. An example is given in lines 28-31 of **index.php**:

```
$smarty = new \Smarty();
$smarty->setTemplateDir(BASE_PATH.'templates');
$smarty->setCompileDir(BASE_PATH.'compile');
$smarty->setCacheDir(BASE_PATH.'compile');
$smarty->setConfigDir(BASE_PATH.'compile');
```

# Step 2 #
Register the custom locale function (if you use it. Again, you don't have to use it if you set your text-domain from within the PHP application AND you've only got one text-domain for all your templates. See [example\_00 here](http://code.google.com/p/smarty-gettext/wiki/SmartyTemplateExamples)). Example given in line 35 of **index.php**:

```
$smarty->registerPlugin('function', 'locale', '\\mebb\\app\\core\\web\\smarty\\functions\\locale');
```

# Step 3 #
Here you choose which files to use for gettext. By default, all templates in your template directory will be used. Example in lines 40, 41 of **index.php**

```
$info = array();//the passback array for error definitions (in case there are any)
$sources = \mebb\lib\i18n\smarty\compile($smarty, null, $info);
```

The **1st parameter** (mandatory) is the Smarty object you created in **Step 1** above.

The **2nd parameter** (optional) of the compile function can be one of the following:
  * null: all templates in the Smarty template directory will be used
  * array: an array of strings, either files and/or directories that should be included into the compilation. They should resolve using the [PHP realpath function](http://php.net/manual/en/function.realpath.php)

The **3rd parameter** (optional) of the compile function is a passback array that contains information about compilation errors etc. It contains the Smarty compilation error message, the file that threw the error and the original exception. See **index.php** lines 62-69 for an example.

The function returns a list of compiled sources.

# Step 4 #
Now you save all the templates into either:
  * a single file: function \mebb\lib\i18n\smarty\save($smarty, $compiled\_sources, $file = null, $empty\_file = true)
  * multiple files: function save\_individual($smarty, $compiled\_sources, $directory = null)

Personally, I'd prefer to use the **save\_individual**, because it preserves the original template file names and allows the [po files](http://www.gnu.org/software/gettext/manual/gettext.html#PO-Files) to contain an understandable reference to the originator of the message-ids

Example excerpt of a PO:
```
#: example_01.tpl:32 example_01.tpl:34 example_01.tpl:36 example_02.tpl:32
#: example_02.tpl:34 example_02.tpl:36 example_03.tpl:32 example_03.tpl:37
#: example_03.tpl:39
#, php-format
msgid "%d comment"
msgid_plural "%d comments"
msgstr[0] "en_GB locale: %d comments"
msgstr[1] "en_GB locale: %d comment"
```

While the line-references created by gettext are not correct you at least get the template name accurately. This helps in large translation projects to establish the translation context.

If you used the **save** method instead, you would only get one generic file-name, e.g.

```
#: /tmp/i18n_Duvq0P:30 /tmp/i18n_Duvq0P:32 /tmp/i18n_Duvq0P:34
#: /tmp/i18n_Duvq0P:67 /tmp/i18n_Duvq0P:69 /tmp/i18n_Duvq0P:71
#: /tmp/i18n_Duvq0P:104 /tmp/i18n_Duvq0P:110 /tmp/i18n_Duvq0P:112
#: /tmp/i18n_Duvq0P:150 /tmp/i18n_Duvq0P:156 /tmp/i18n_Duvq0P:158
#: /tmp/i18n_Duvq0P:194 /tmp/i18n_Duvq0P:200 /tmp/i18n_Duvq0P:202
#, php-format
msgid "%d comment"
msgid_plural "%d comments"
msgstr[0] ""
msgstr[1] ""
```

That's basically it :) You're done...

# Now what? Extract PO files #
When calling save or save\_individual you either can pass them a path to a file/directory or it will automatically create one in the OS temporary directory and return it for you.

All that's left to do is calling e.g. xgettext:

if you called **save\_indivdual**

  * cd DIRECTORY
  * xgettext -n `*`.tpl --language=PHP

if you called **save**

  * xgettext -n PATH\_TO\_FILE --language=PHP


The **--language=PHP** flag is crucial since it enables xgettext to reliably detect the files to be processed as PHP files, regardless of the extension.

Now you've got your messages.po file and you can take it from there.

Have a look at the [GNU gettext Manual](http://www.gnu.org/software/gettext/manual/gettext.html#PO-Files) where to take it from here.