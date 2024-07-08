## Directory Structure
- oxideshop (SHOP_ROOT)
	- extensions (MODULES_FOLDER)
	- phpqa
		- bin
			- [[#ide-helper.php]]
		- [[#composer.json]]
	- source (WEB_ROOT)
		- index.php
		- confic.inc.php
	- [[#.php-cs-fixer.dist.php]]
	- [[#.psalm-autoload.php]]
	- [[#composer.json]]
	- [[#phpcs.xml]]
	- [[#phpmd.xml]]
	- [[#phpstan.neon]]
	- [[#phpstan-baseline.neon]] (optional)
	- [[#psalm.xml]]
	- [[#rector.php]]


## PHPQA-Folder
In order to avoid conflicts with dependencies from our shop, it is recommended that you put your quality tools in a separated folder with its own composer.json
#### ide-helper.php
Copy the content of this link:
https://raw.githubusercontent.com/proudcommerce/oxid-ide-modulehelper/master/pc-ide-helper.php

Save the content in this file: `SHOP_ROOT/phpqa/bin/ide-helper.php`
(We are working on a pull request to make that file require-able via composer)

**Important** Overwrite these lines with the following content:
`Line 61`:

```php
$modulePath = dirname(__DIR__, 2) . DIRECTORY_SEPARATOR . 'extensions';
```
`Line 72`:

```php
$modulePath = $helperFile = dirname(__FILE__, 2) . DIRECTORY_SEPARATOR . '.module-helper.php';
```
Please adjust "extensions" to your MODULES_FOLDER
#### composer.json

Copy this content to `SHOP_ROOT/phpqa/composer.json`:
```json
{  
    "require": {  
        "php": ">=8.0",  
        "friendsofphp/php-cs-fixer": "^v3.39.0",  
        "php-parallel-lint/php-parallel-lint": "^v1.3.2",  
        "phpmd/phpmd": "^2.14.1",  
        "phpstan/phpstan": "^1.10.44",  
        "squizlabs/php_codesniffer": "^3.7.2",  
        "vimeo/psalm": "^5.16.0",  
        "rector/rector": "^0.17.13"  
    },  
    "config": {  
       "preferred-install": {  
           "*": "dist"  
       }  
    }  
}
```

You could require these packages by yourself to get the most updated versions. We just showing our composer.json file here, because it is tested with Oxid 7.
If you want to require them yourself, you can run this command:
```bash
composer require friendsofphp/php-cs-fixer php-parallel-lint/php-parallel-lint phpmd/phpmd phpstan/phpstan squizlabs/php_codesniffer vimeo/psalm ector/rector
```

**Important** Make sure that you run this command within the `phpqa` folder, not the `SHOP_ROOT`

## SHOP_ROOT-Folder
### .php-cs-fixer.dist.php
Example content
```php
<?php  
  
declare(strict_types=1);  
  
$year = date('Y');  
$header = <<<EOF  
This Software is the property of OXID eSales and is protected  
by copyright law - it is NOT Freeware.  
  
Any unauthorized use of this software without a valid license key  
is a violation of the license agreement and will be prosecuted by  
civil and criminal law.  
  
@author          OXID Solution Catalysts  
@link            https://www.oxid-esales.com  
@copyright (C)   OXID eSales AG 2003-$year  
EOF;  
  
$finder = PhpCsFixer\Finder::create()  
    ->files()  
    ->in(__DIR__ . '/extensions')  
    //->exclude('excluded_module')  
;  
  
$config = new PhpCsFixer\Config();  
return $config  
    ->setFinder($finder)  
    ->setRiskyAllowed(true)  
    ->setRules([  
        '@PSR12' => true,  
        '@PSR12:risky' => true,  
        '@PHP80Migration' => true,  
        '@PHP80Migration:risky' => false,  
        '@PhpCsFixer' => true,  
        '@PhpCsFixer:risky' => false,  
        'header_comment' => ['header' => $header, 'comment_type' => 'PHPDoc', 'location' => 'after_open'],  
        'phpdoc_no_empty_return' => false,  
        'binary_operator_spaces' => ['operators' => ['=>' => 'align_single_space_minimal']],  
        'concat_space' => ['spacing' => 'one'],  
        'declare_strict_types' => true,  
        'global_namespace_import' => ['import_classes' => true, 'import_constants' => true, 'import_functions' => true],  
        'increment_style' => ['style' => 'post'],  
        'single_line_empty_body' => false,  
        'types_spaces' => ['space_multiple_catch' => 'single'],  
        'yoda_style' => ['equal' => false, 'identical' => false, 'less_and_greater' => false],  
    ]);
```

**Important** Make sure to adjust the content of the `$header` variable. This content will be pre-pended to all *.php Files

**Important** Please adjust the `->in(__DIR__ . '/extensions')` to your needs. You need to point to the directory where your modules are saved.  If you want to exclude directories within the modules-folder, you can name them via `->exclude()` method. The path must be absolute or relative to the path of `->in()`

### .psalm-autoload.php
Example content
```php
<?php  
  
require_once '.module-helper.php';  
require_once 'source/oxfunctions.php';  
require_once 'source/overridablefunctions.php';  
require_once 'vendor/oxid-esales/oxideshop-ce/source/Core/Model/BaseModel.php';  
require_once 'vendor/autoload.php';
```

If you need to include additional files which should be loaded, before psalm runs, please extend this list here.

### phpcs.xml
```xml
<?xml version="1.0"?>  
<ruleset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="PHP_CodeSniffer" xsi:noNamespaceSchemaLocation="phpcs.xsd">  
    <file>./extensions</file>    
    <!--<exclude-pattern>./extensions/excluded_module</exclude-pattern>-->
  
    <arg name="extensions" value="php"/>  
  
    <rule ref="PSR12"/>  
    <rule ref="Generic.PHP.RequireStrictTypes"/>  
</ruleset>
```

**Important** Make sure to adjust the path to your `MODULES_FOLDER`

### phpmd.xml
```xml
<?xml version="1.0"?>  
<ruleset name="Ruleset for BWB"  
         xmlns="http://pmd.sf.net/ruleset/1.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://pmd.sf.net/ruleset/1.0.0  
                     http://pmd.sf.net/ruleset_xml_schema.xsd"         xsi:noNamespaceSchemaLocation="  
                     http://pmd.sf.net/ruleset_xml_schema.xsd">  
    <description>  
        Ruleset for PROJECT_NAME
    </description>  
  
    <rule ref="rulesets/codesize.xml" />  
    <rule ref="rulesets/design.xml" />  
    <rule ref="rulesets/naming.xml" />
    <rule ref="rulesets/unusedcode.xml" />
    <rule ref="rulesets/cleancode.xml" />  
	<!--
	<exclude-pattern>extensions/exluded_module</exclude-pattern>
	-->
  
</ruleset>
```
We don't need to specify where our modules are

**Important** Adjust the path for the `exclude` element, if needed

### phpstan.neon
Example
```yaml
#includes:  
#    - phpstan-baseline.neon  
  
parameters:  
    level: 5
    paths:  
        - extensions
#    excludePaths:  
#        - extensions/excluded_module
    scanFiles:  
        - source/oxfunctions.php  
        - source/overridablefunctions.php  
        - source/bootstrap.php  
    bootstrapFiles:  
        - .module-helper.php  
    parallel:  
        processTimeout: 600.0  
    earlyTerminatingMethodCalls:  
        OxidSolutionCatalysts\UtilityComponent\Service\DateTime\DateTimeService:  
            - throwDateTimeZoneException
```

**Important** Make sure to adjust the path to your `MODULES_FOLDER`

**About the first two lines (the baseline-file)**
We will dive into that later, when we setup the commands for our quality tools.

### phpstan-baseline.neon
```yaml
parameters:  
    ignoreErrors:
```
It's only the needed basic-content of the file. It will be filled later by phpstan

### psalm.xml
Example
```xml
<?xml version="1.0"?>  
<psalm  
    autoloader=".psalm-autoload.php"  
    errorLevel="4"  
    findUnusedBaselineEntry="true"  
    findUnusedCode="false"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns="https://getpsalm.org/schema/config"  
    xsi:schemaLocation="https://getpsalm.org/schema/config vendor/vimeo/psalm/config.xsd"  
>  
    <projectFiles>  
        <directory name="source/modules/project"/>  
        <ignoreFiles>  
            <directory name="extensions" />
            <!--
            <ignoreFiles>  
			    <directory name="extensions/excluded_module" />
	        </ignoreFiles>  
	        -->
    </projectFiles>  
</psalm>
```

**Important** Make sure to adjust the path to your `MODULES_FOLDER`

### rector.php
```php
<?php  
  
declare(strict_types=1);  
  
use Rector\Config\RectorConfig;  
use Rector\Php74\Rector\LNumber\AddLiteralSeparatorToNumberRector;  
use Rector\Set\ValueObject\LevelSetList;  
  
return static function (RectorConfig $rectorConfig): void {  
    $rectorConfig->paths(  
        [  
            __DIR__ . '/extensions',  
        ]  
    );
    /*
    $rectorConfig->skip(  
        [  
            // Exlude pathes, comma separated
            __DIR__ . '/extensions/excluded_module',
            // Exlude Rector-Rules
            AddLiteralSeparatorToNumberRector::class,  
        ]  
    );
    */  
    $rectorConfig->import(LevelSetList::UP_TO_PHP_80);  
    $rectorConfig->parallel(600, 16, 20);  
    $rectorConfig->phpstanConfig(__DIR__ . '/phpstan.neon');  
};
```

**Important** Make sure to adjust the path to your `MODULES_FOLDER`

### composer.json
Please install first the oxideshop-ide-helper:
```bash
composer require --dev oxid-esales/oxideshop-ide-helper
```

We will now add some scripts to our `composer.json` to run our quality tools easily from the `SHOP_ROOT` folder.
You can keep your remaining file as it is, you just need to modify/append the `scripts` array:
```json
"scripts": {
        "post-install-cmd": [
            "@osc:phpqa:install",
            "@oe:ide-helper:generate",
            "@osc:ide-helper:generate"
        ],
        "post-update-cmd": [
            "@oe:ide-helper:generate",
            "@osc:ide-helper:generate"
        ],
        "oe:ide-helper:generate": [
            "if [ -f ./vendor/bin/oe-eshop-ide_helper ]; then oe-eshop-ide_helper; fi"
        ],
        "osc:ide-helper:generate": [
            "php -d xdebug.mode=off phpqa/bin/ide-helper.php"
        ],
        "osc:phpqa:install": "vendor/bin/composer install --working-dir=phpqa --prefer-dist",
        "osc:phpqa:update": "vendor/bin/composer update --working-dir=phpqa",
        "osc:phpqa:rector": "XDEBUG_MODE=off \\phpqa/vendor/bin/rector --dry-run",
        "osc:phpqa:rector:fix": "XDEBUG_MODE=off \\phpqa/vendor/bin/rector",
        "osc:phpqa:linter": "XDEBUG_MODE=off \\phpqa/vendor/bin/parallel-lint extensions ",
        "osc:phpqa:fixer": "XDEBUG_MODE=off \\phpqa/vendor/bin/php-cs-fixer fix --dry-run",
        "osc:phpqa:fixer:fix": "XDEBUG_MODE=off \\phpqa/vendor/bin/php-cs-fixer fix",
        "osc:phpqa:phpcs": "XDEBUG_MODE=off \\phpqa/vendor/bin/phpcs",
        "osc:phpqa:phpmd": "XDEBUG_MODE=off \\phpqa/vendor/bin/phpmd source/modules/project ansi phpmd.xml",
        "osc:phpqa:phpmd:baseline": "\\phpqa/vendor/bin/phpmd source/modules/project/ ansi phpmd.xml --generate-baseline",
        "osc:phpqa:psalm": "XDEBUG_MODE=off \\phpqa/vendor/bin/psalm --no-cache --no-suggestions",
        "osc:phpqa:psalm:security": "XDEBUG_MODE=off \\phpqa/vendor/bin/psalm --taint-analysis --no-cache",
        "osc:phpqa:phpstan": "XDEBUG_MODE=off \\phpqa/vendor/bin/phpstan analyse --memory-limit=1G --configuration phpstan.neon",
        "osc:phpqa:phpstan:baseline": "XDEBUG_MODE=off \\phpqa/vendor/bin/phpstan analyse --memory-limit=1G --configuration phpstan.neon --generate-baseline",
        "osc:phpqa": [
            "@osc:ide-helper:generate",
            "@osc:phpqa:rector",
            "@osc:phpqa:linter",
            "@osc:phpqa:fixer",
            "@osc:phpqa:phpcs",
            "@osc:phpqa:psalm",
            "@osc:phpqa:psalm:security",
            "@osc:phpqa:phpstan"
        ]
    }
```


A little bit of an explanation what we are doing here.
Many commands should be self-explaining, but I want to point out a few of them.

#### post-install-cmd / post-update-cmd
We want to make sure that the ide/module helper files are generated when we install/update modules.

#### XDEBUG_MODE=off
Some quality tools will complain when the xdebug mode is enabled, because it will reduce their performance. To avoid the annoying warning and to keep the performance, we have prefixed all commands with this.

#### osc:phpqa:linter
Make sure to replace `extensions` with your `MODULES_FOLDER`

If you want to exclude folder like you may did in some of the configuration files, please note them in this line directly. You can list the `--exlude` parameter multiple times
Example:
```bash
"osc:phpqa:linter": "XDEBUG_MODE=off \\phpqa/vendor/bin/parallel-lint extensions --exlude extensions/excluded_module_onne --exclude extensions/excluded_module_two"
```
#### :fix
We have some commands which exists to times. With `:fix` at the end and without it.
It's highly recommended that you run the commands without `:fix`first to review the error, which will be automatically fixed when you run the command with `:fix`

### suggested practice
To run the commands, you can just call `composer run [SCRIPT_NAME]` from your `SHOP_ROOT` where script name is the index of the scripts array.
Example:
```bash
comopser run osc:phpqa:linter
```

You should run each script from the `osc:phpqa`array one by one. You could call the whole block together like this:
```bash
comopser run osc:phpqa
```

**Disadvantage**
The block will stop as soon as one of the scripts throws an error. When you have fixed the error, you would need to call the block again. But then you will always call quality tools which may have already run successfully.

That's why you should run them one by one.
When a tool reports an error, you can review the report and may run `:fix` to fix the problems manually or you fix the errors manually and run that specific tool again.

**Why do we have the block then?**
The block can be used to integrate these tools into your CI pipeline.The pipeline will break as soon as one tool encounters an error. That's how you can make sure, that no non-quality-checked  branches will be merged.


#### osc:phpqa:phpmd:baseline / osc:phpqa:phpstan:baseline
**What is a baseline?**
Sometimes you will encounter problems in your code which are false-positive reported from the quality tools. Some problems are maybe also not-fixable.

**Example**
When you extend a Oxid-Core-Class by adding an additional method to it, you will get a report from `phpstan`, that the method may not exists. That is caused by our class-extension-chain. To avoid this, you have two options:

**Option 1:** Bloat the code
You could tell phpstan above all problematic lines that the next line should be ignored: 
`// @phpstan-ignore-next-line`.

**Option 2:** Use the Baseline
When you have reviewed your error report and think that all remaining errors are "not-fixable", you can generate the baseline file.
In this file, the tool will safe all "known" errors which will be ignored the next time, the tool is run.
