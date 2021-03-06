# PHPExcel Developer Documentation

# Creating a spreadsheet

## The PHPExcel class

The PHPExcel class is the core of PHPExcel. It contains references to the contained worksheets, document security settings and document meta data.

To simplify the PHPExcel concept: the PHPExcel class represents your workbook.

Typically, you will create a workbook in one of two ways, either by loading it from a spreadsheet file, or creating it manually. A third option, though less commonly used, is cloning an existing workbook that has been created using one of the previous two methods.

### Loading a Workbook from a file

Details of the different spreadsheet formats supported, and the options available to read them into a PHPExcel object are described fully in the “PHPExcel User Documentation - Reading Spreadsheet Files” document.


$inputFileName� =� './sampleData/example1.xls';

/**� Load� $inputFileName� to� a� PHPExcel� Object� � **/
$objPHPExcel� =� PHPExcel_IOFactory::load($inputFileName);


### Creating a new workbook

If you want to create a new workbook, rather than load one from file, then you simply need to instantiate it as a new PHPExcel object.


/**� Create a� new PHPExcel� Object� � **/
$objPHPExcel� =� new PHPExcel();


A new workbook will always be created with a single worksheet.

## Configuration Settings

Once you have included the PHPExcel files in your script, but before instantiating a PHPExcel object or loading a workbook file, there are a number of configuration options that can be set which will affect the subsequent behaviour of the script.

### Cell Caching

PHPExcel uses an average of about 1k/cell in your worksheets, so large workbooks can quickly use up available memory. Cell caching provides a mechanism that allows PHPExcel to maintain the cell objects in a smaller size of memory, on disk, or in APC, memcache or Wincache, rather than in PHP memory. This allows you to reduce the memory usage for large workbooks, although at a cost of speed to access cell data.

By default, PHPExcel still holds all cell objects in memory, but you can specify alternatives. To enable cell caching, you must call the PHPExcel_Settings::setCacheStorageMethod() method, passing in the caching method that you wish to use.

$cacheMethod = PHPExcel_CachedObjectStorageFactory::cache_in_memory;

PHPExcel_Settings::setCacheStorageMethod($cacheMethod);

setCacheStorageMethod() will return a boolean true on success, false on failure (for example if trying to cache to APC when APC is not enabled).

A separate cache is maintained for each individual worksheet, and is automatically created when the worksheet is instantiated based on the caching method and settings that you have configured. You cannot change the configuration settings once you have started to read a workbook, or have created your first worksheet.

Currently, the following caching methods are available.

PHPExcel_CachedObjectStorageFactory::cache_in_memory;

The default. If you don’t initialise any caching method, then this is the method that PHPExcel will use. Cell objects are maintained in PHP memory as at present.

PHPExcel_CachedObjectStorageFactory::cache_in_memory_serialized;

Using this caching method, cells are held in PHP memory as an array of serialized objects, which reduces the memory footprint with minimal performance overhead.

PHPExcel_CachedObjectStorageFactory::cache_in_memory_gzip;

Like cache_in_memory_serialized, this method holds cells in PHP memory as an array of serialized objects, but gzipped to reduce the memory usage still further, although access to read or write a cell is slightly slower.

PHPExcel_CachedObjectStorageFactory::cache_igbinary;

Uses PHP’s igbinary extension (if it’s available) to serialize cell objects in memory. This is normally faster and uses less memory than standard PHP serialization, but isn’t available in most hosting environments.

PHPExcel_CachedObjectStorageFactory::cache_to_discISAM;

When using cache_to_discISAM all cells are held in a temporary disk file, with only an index to their location in that file maintained in PHP memory. This is slower than any of the cache_in_memory methods, but significantly reduces the memory footprint. By default, PHPExcel will use PHP’s temp directory for the cache file, but you can specify a different directory when initialising cache_to_discISAM.

$cacheMethod = PHPExcel_CachedObjectStorageFactory:: cache_to_discISAM;

$cacheSettings = array( 'dir'  => '/usr/local/tmp'

);

PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);

The temporary disk file is automatically deleted when your script terminates.

PHPExcel_CachedObjectStorageFactory::cache_to_phpTemp;

Like cache_to_discISAM, when using cache_to_phpTemp all cells are held in the php://temp I/O stream, with only an index to their location maintained in PHP memory. In PHP, the php://memory wrapper stores data in the memory: php://temp behaves similarly, but uses a temporary file for storing the data when a certain memory limit is reached. The default is 1 MB, but you can change this when initialising cache_to_phpTemp.

$cacheMethod = PHPExcel_CachedObjectStorageFactory:: cache_to_phpTemp;

$cacheSettings = array( 'memoryCacheSize'  => '8MB'

);

PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);

The php://temp file is automatically deleted when your script terminates.

PHPExcel_CachedObjectStorageFactory::cache_to_apc;

When using cache_to_apc, cell objects are maintained in APC with only an index maintained in PHP memory to identify that the cell exists. By default, an APC cache timeout of 600 seconds is used, which should be enough for most applications: although it is possible to change this when initialising cache_to_APC.

$cacheMethod = PHPExcel_CachedObjectStorageFactory::cache_to_APC;

$cacheSettings = array( 'cacheTime'        => 600

);

PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);

When your script terminates all entries will be cleared from APC, regardless of the cacheTime value, so it cannot be used for persistent storage using this mechanism.

PHPExcel_CachedObjectStorageFactory::cache_to_memcache

When using cache_to_memcache, cell objects are maintained in memcache with only an index maintained in PHP memory to identify that the cell exists.

By default, PHPExcel looks for a memcache server on localhost at port 11211. It also sets a memcache timeout limit of 600 seconds. If you are running memcache on a different server or port, then you can change these defaults when you initialise cache_to_memcache:

$cacheMethod = PHPExcel_CachedObjectStorageFactory::cache_to_memcache;

$cacheSettings = array( 'memcacheServer'  => 'localhost',

'memcachePort'    => 11211,

'cacheTime'       => 600

);

PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);

When your script terminates all entries will be cleared from memcache, regardless of the cacheTime value, so it cannot be used for persistent storage using this mechanism.

PHPExcel_CachedObjectStorageFactory::cache_to_wincache;

When using cache_to_wincache, cell objects are maintained in Wincache with only an index maintained in PHP memory to identify that the cell exists. By default, a Wincache cache timeout of 600 seconds is used, which should be enough for most applications: although it is possible to change this when initialising cache_to_wincache.

$cacheMethod = PHPExcel_CachedObjectStorageFactory::cache_to_wincache;

$cacheSettings = array( 'cacheTime'        => 600

);

PHPExcel_Settings::setCacheStorageMethod($cacheMethod, $cacheSettings);

When your script terminates all entries will be cleared from Wincache, regardless of the cacheTime value, so it cannot be used for persistent storage using this mechanism.

PHPExcel_CachedObjectStorageFactory::cache_to_sqlite;

Uses an SQLite 2 in-memory database for caching cell data. Unlike other caching methods, neither cells nor an index are held in PHP memory - an indexed database table makes it unnecessary to hold any index in PHP memory – making this the most memory-efficient of the cell caching methods.

PHPExcel_CachedObjectStorageFactory::cache_to_sqlite3;

Uses an SQLite 3 in-memory database for caching cell data. Unlike other caching methods, neither cells nor an index are held in PHP memory - an indexed database table makes it unnecessary to hold any index in PHP memory – making this the most memory-efficient of the cell caching methods.

### Language/Locale

Some localisation elements have been included in PHPExcel. You can set a locale by changing the settings. To set the locale to Brazilian Portuguese you would use:

$locale = 'pt_br';

$validLocale = PHPExcel_Settings::setLocale($locale);

if (!$validLocale) {

echo 'Unable to set locale to '.$locale." - reverting to en_us<br />\n";

}

If Brazilian Portuguese language files aren’t available, then the Portuguese will be enabled instead: if Portuguese language files aren’t available, then the setLocale() method will return an error, and American English (en_us) settings will be used throughout.

More details of the features available once a locale has been set, including a list of the languages and locales currently supported, can be found in section  REF _Ref259954895 \r \h 4.5.5  REF _Ref259954904 \h Locale Settings for Formulae.

## Clearing a Workbook from memory

The PHPExcel object contains cyclic references (e.g. the workbook is linked to the worksheets, and the worksheets are linked to their parent workbook) which cause problems when PHP tries to clear the objects from memory when they are unset(), or at the end of a function when they are in local scope. The result of this is “memory leaks”, which can easily use a large amount of PHP’s limited memory.

This can only be resolved manually: if you need to unset a workbook, then you also need to “break” these cyclic references before doing so. PHPExcel provides the disconnectWorksheets() method for this purpose.

$objPHPExcel->disconnectWorksheets();

unset($objPHPExcel);

## Worksheets

A worksheet is a collection of cells, formula’s, images, graphs, … It holds all data necessary to represent as a spreadsheet worksheet.

When you load a workbook from a spreadsheet file, it will be loaded with all its existing worksheets (unless you specified that only certain sheets should be loaded). When you load from non-spreadsheet files (such as a CSV or HTML file) or from spreadsheet formats that don’t identify worksheets by name (such as SYLK), then a single worksheet called “WorkSheet” will be created containing the data from that file.

When you instantiate a new workbook, PHPExcel will create it with a single worksheet called “WorkSheet”.

The getSheetCount() method will tell you the number of worksheets in the workbook; while the getSheetNames() method will return a list of all worksheets in the workbook, indexed by the order in which their “tabs” would appear when opened in MS Excel (or other appropriate Spreadsheet program).

Individual worksheets can be accessed by name, or by their index position in the workbook. The index position represents the order that each worksheet “tab” is shown when the workbook is opened in MS Excel (or other appropriate Spreadsheet program). To access a sheet by its index, use the getSheet() method.

// Get the second sheet in the workbook

// Note that sheets are indexed from 0

$objPHPExcel->getSheet(1);

If you don’t specify a sheet index, then the first worksheet will be returned.

Methods also exist allowing you to reorder the worksheets in the workbook.

To access a sheet by name, use the getSheetByName() method, specifying the name of the worksheet that you want to access.

// Retrieve the worksheet called 'Worksheet 1'

$objPHPExcel->getSheetByName('Worksheet 1');

Alternatively, one worksheet is always the currently active worksheet, and you can access that directly. The currently active worksheet is the one that will be active when the workbook is opened in MS Excel (or other appropriate Spreadsheet program).

// Retrieve the current active worksheet

$objPHPExcel->getActiveSheet();

You can change the currently active sheet by index or by name using the setActiveSheetIndex() and setActiveSheetIndexByName()methods.

### Adding a new Worksheet

You can add a new worksheet to the workbook using the createSheet() method of the PHPExcel object. By default, this will be created as a new “last” sheet; but you can also specify an index position as an argument, and the worksheet will be inserted at that position, shuffling all subsequent worksheets in the collection down a place.

$objPHPExcel->createSheet();

A new worksheet created using this method will be called “Worksheet” or “Worksheet<n>” where “<n>” is the lowest number possible to guarantee that the title is unique.

Alternatively, you can instantiate a new worksheet (setting the title to whatever you choose) and then insert it into your workbook using the addSheet() method.

// Create a new worksheet called “My Data”

$myWorkSheet = new PHPExcel_Worksheet($objPHPExcel, 'My Data');

// Attach the “My Data” worksheet as the first worksheet in the PHPExcel object

$objPHPExcel->addSheet($myWorkSheet, 0);

If you don’t specify an index position as the second argument, then the new worksheet will be added after the last existing worksheet.

### Copying Worksheets

Sheets within the same workbook can be copied by creating a clone of the worksheet you wish to copy, and then using the addSheet() method to insert the clone into the workbook.

$objClonedWorksheet = clone $objPHPExcel->getSheetByName('Worksheet 1');

$objClonedWorksheet->setTitle('Copy of Worksheet 1')

$objPHPExcel->addSheet($objClonedWorksheet);

You can also copy worksheets from one workbook to another, though this is more complex as PHPExcel also has to replicate the styling between the two workbooks. The addExternalSheet() method is provided for this purpose.

$objClonedWorksheet = clone $objPHPExcel1->getSheetByName('Worksheet 1');

$objPHPExcel->addExternalSheet($objClonedWorksheet);

In both cases, it is the developer’s responsibility to ensure that worksheet names are not duplicated. PHPExcel will throw an exception if you attempt to copy worksheets that will result in a duplicate name.

### Removing a Worksheet

You can delete a worksheet from a workbook, identified by its index position, using the removeSheetByIndex() method

$sheetIndex = $objPHPExcel->getIndex($objPHPExcel-> getSheetByName('Worksheet 1'));

$objPHPExcel->removeSheetByIndex($sheetIndex);

If the currently active worksheet is deleted, then the sheet at the previous index position will become the currently active sheet.

## Accessing cells

Accessing cells in a PHPExcel worksheet should be pretty straightforward. This topic lists some of the options to access a cell.

### Setting a cell value by coordinate

Setting a cell value by coordinate can be done using the worksheet’s setCellValue method.

$objPHPExcel->getActiveSheet()->setCellValue('B8', 'Some value');

### Retrieving a cell by coordinate

To retrieve the value of a cell, the cell should first be retrieved from the worksheet using the getCell method. A cell’s value can be read again using the following line of code:

$objPHPExcel->getActiveSheet()->getCell('B8')->getValue();

If you need the calculated value of a cell, use the following code. This is further explained in  REF _Ref191885372 \w \h  \* MERGEFORMAT 4.4.35.

$objPHPExcel->getActiveSheet()->getCell('B8')->getCalculatedValue();

### Setting a cell value by column and row

Setting a cell value by coordinate can be done using the worksheet’s setCellValueByColumnAndRow method.

// Set cell B8
$objPHPExcel->getActiveSheet()->setCellValueByColumnAndRow(1, 8, 'Some value');

### Retrieving a cell by column and row

To retrieve the value of a cell, the cell should first be retrieved from the worksheet using the getCellByColumnAndRow method. A cell’s value can be read again using the following line of code:

// Get cell B8
$objPHPExcel->getActiveSheet()->getCellByColumnAndRow(1, 8)->getValue();

If you need the calculated value of a cell, use the following code. This is further explained in  REF _Ref191885372 \w \h 4.4.35

// Get cell B8
$objPHPExcel->getActiveSheet()->getCellByColumnAndRow(1, 8)->getCalculatedValue();

### Looping cells

#### Looping cells using iterators

The easiest way to loop cells is by using iterators. Using iterators, one can use foreach to loop worksheets, rows and cells.

Below is an example where we read all the values in a worksheet and display them in a table.

<?php

$objReader = PHPExcel_IOFactory::createReader('Excel2007');

$objReader->setReadDataOnly(true);

$objPHPExcel = $objReader->load("test.xlsx");

$objWorksheet = $objPHPExcel->getActiveSheet();

echo '<table>' . "\n";

foreach ($objWorksheet->getRowIterator() as $row) {

echo '<tr>' . "\n";

$cellIterator = $row->getCellIterator();

$cellIterator->setIterateOnlyExistingCells(false); // This loops all cells,
                                                     // even if it is not set.
                                                     // By default, only cells
                                                     // that are set will be
                                                     // iterated.

foreach ($cellIterator as $cell) {

echo '<td>' . $cell->getValue() . '</td>' . "\n";

}

echo '</tr>' . "\n";

}

echo '</table>' . "\n";
?>

Note that we have set the cell iterator’s setIterateOnlyExistingCells() to false. This makes the iterator loop all cells, even if they were not set before.

__The cell iterator will return ____null____ as the cell if it is not set in the worksheet.__
Setting the cell iterator’s setIterateOnlyExistingCells()to false will loop all cells in the worksheet that can be available at that moment. This will create new cells if required and increase memory usage! Only use it if it is intended to loop all cells that are possibly available.

#### Looping cells using indexes

One can use the possibility to access cell values by column and row index like (0,1) instead of 'A1' for reading and writing cell values in loops.

Note: In PHPExcel column index is 0-based while row index is 1-based. That means 'A1' ~ (0,1)

Below is an example where we read all the values in a worksheet and display them in a table.

<?php

$objReader = PHPExcel_IOFactory::createReader('Excel2007');

$objReader->setReadDataOnly(true);

$objPHPExcel = $objReader->load("test.xlsx");

$objWorksheet = $objPHPExcel->getActiveSheet();

$highestRow = $objWorksheet->getHighestRow(); // e.g. 10

$highestColumn = $objWorksheet->getHighestColumn(); // e.g 'F'

$highestColumnIndex = PHPExcel_Cell::columnIndexFromString($highestColumn); // e.g. 5

echo '<table>' . "\n";

for ($row = 1; $row <= $highestRow; ++$row) {

echo '<tr>' . "\n";

for ($col = 0; $col <= $highestColumnIndex; ++$col) {

echo '<td>' . $objWorksheet->getCellByColumnAndRow($col, $row)->getValue() . '</td>' . "\n";

}

echo '</tr>' . "\n";

}

echo '</table>' . "\n";

?>

### Using value binders to facilitate data entry

Internally, PHPExcel uses a default PHPExcel_Cell_IValueBinder implementation (PHPExcel_Cell_DefaultValueBinder) to determine data types of entered data using a cell’s setValue() method.

Optionally, the default behaviour of PHPExcel can be modified, allowing easier data entry. For example, a PHPExcel_Cell_AdvancedValueBinder class is present. It automatically converts percentages and dates entered as strings to the correct format, also setting the cell’s style information. The following example demonstrates how to set the value binder in PHPExcel:

/** PHPExcel */

require_once 'PHPExcel.php';

/** PHPExcel_Cell_AdvancedValueBinder */

require_once 'PHPExcel/Cell/AdvancedValueBinder.php';

/** PHPExcel_IOFactory */

require_once 'PHPExcel/IOFactory.php';

// Set value binder

PHPExcel_Cell::setValueBinder( new PHPExcel_Cell_AdvancedValueBinder() );

// Create new PHPExcel object

$objPHPExcel = new PHPExcel();

// ...

// Add some data, resembling some different data types

$objPHPExcel->getActiveSheet()->setCellValue('A4', 'Percentage value:');

$objPHPExcel->getActiveSheet()->setCellValue('B4', '10%');
              // Converts to 0.1 and sets percentage cell style

$objPHPExcel->getActiveSheet()->setCellValue('A5', 'Date/time value:');

$objPHPExcel->getActiveSheet()->setCellValue('B5', '21 December 1983');
              // Converts to date and sets date format cell style

__Creating your own value binder is easy____.__
When advanced value binding is required, you can implement the PHPExcel_Cell_IValueBinder interface or extend the PHPExcel_Cell_DefaultValueBinder or PHPExcel_Cell_AdvancedValueBinder classes.

## PHPExcel recipes

The following pages offer you some widely-used PHPExcel recipes. Please note that these do NOT offer complete documentation on specific PHPExcel API functions, but just a bump to get you started. If you need specific API functions, please refer to the API documentation.

For example,  REF _Ref191885321 \w \h 4.4.7  REF _Ref191885321 \h Setting a worksheet’s page orientation and size covers setting a page orientation to A4. Other paper formats, like US Letter, are not covered in this document, but in the PHPExcel API documentation.

### Setting a spreadsheet’s metadata

PHPExcel allows an easy way to set a spreadsheet’s metadata, using document property accessors. Spreadsheet metadata can be useful for finding a specific document in a file repository or a document management system. For example Microsoft Sharepoint uses document metadata to search for a specific document in its document lists.

Setting spreadsheet metadata is done as follows:

$objPHPExcel->getProperties()->setCreator("Maarten Balliauw");

$objPHPExcel->getProperties()->setLastModifiedBy("Maarten Balliauw");

$objPHPExcel->getProperties()->setTitle("Office 2007 XLSX Test Document");

$objPHPExcel->getProperties()->setSubject("Office 2007 XLSX Test Document");

$objPHPExcel->getProperties()->setDescription("Test document for Office 2007 XLSX, generated using PHP classes.");

$objPHPExcel->getProperties()->setKeywords("office 2007 openxml php");

$objPHPExcel->getProperties()->setCategory("Test result file");

### Setting a spreadsheet’s active sheet

The following line of code sets the active sheet index to the first sheet:

$objPHPExcel->setActiveSheetIndex(0);

### Write a date or time into a cell

In Excel, dates and Times are stored as numeric values counting the number of days elapsed since 1900-01-01. For example, the date '2008-12-31' is represented as 39813. You can verify this in Microsoft Office Excel by entering that date in a cell and afterwards changing the number format to 'General' so the true numeric value is revealed. Likewise, '3:15 AM' is represented as 0.135417.

PHPExcel works with UST (Universal Standard Time) date and Time values, but does no internal conversions; so it is up to the developer to ensure that values passed to the date/time conversion functions are UST.

Writing a date value in a cell consists of 2 lines of code. Select the method that suits you the best. Here are some examples:

/* PHPExcel_Cell_AdvanceValueBinder required for this sample */

require_once 'PHPExcel/Cell/AdvancedValueBinder.php';

// MySQL-like timestamp '2008-12-31' or date string

PHPExcel_Cell::setValueBinder( new PHPExcel_Cell_AdvancedValueBinder() );

$objPHPExcel->getActiveSheet()

->setCellValue('D1', '2008-12-31');

$objPHPExcel->getActiveSheet()

->getStyle('D1')

->getNumberFormat()

->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_DATE_YYYYMMDDSLASH)

// PHP-time (Unix time)

$time = gmmktime(0,0,0,12,31,2008); // int(1230681600)

$objPHPExcel->getActiveSheet()

->setCellValue('D1', PHPExcel_Shared_Date::PHPToExcel($time));

$objPHPExcel->getActiveSheet()

->getStyle('D1')

->getNumberFormat()

->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_DATE_YYYYMMDDSLASH)

// Excel-date/time

$objPHPExcel->getActiveSheet()

->setCellValue('D1', 39813)

$objPHPExcel->getActiveSheet()

->getStyle('D1')

->getNumberFormat()

->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_DATE_YYYYMMDDSLASH)

The above methods for entering a date all yield the same result. PHPExcel_Style_NumberFormat provides a lot of pre-defined date formats.

The PHPExcel_Shared_Date::PHPToExcel() method will also work with a PHP DateTime object.

Similarly, times (or date and time values) can be entered in the same fashion: just remember to use an appropriate format code.

__Notes:__

See section "Using value binders to facilitate data entry" to learn more about the AdvancedValueBinder used in the first example.
In previous versions of PHPExcel up to and including 1.6.6, when a cell had a date-like number format code, it was possible to enter a date directly using an integer PHP-time without converting to Excel date format. Starting with PHPExcel 1.6.7 this is no longer supported.
Excel can also operate in a 1904-based calendar (default for workbooks saved on Mac). Normally, you do not have to worry about this when using PHPExcel.### Write a formula into a cell

Inside the Excel file, formulas are always stored as they would appear in an English version of Microsoft Office Excel, and PHPExcel handles all formulae internally in this format. This means that the following rules hold:

Decimal separator is '.' (period)Function argument separator is ',' (comma)Matrix row separator is ';' (semicolon)English function names must be usedThis is regardless of which language version of Microsoft Office Excel may have been used to create the Excel file.

When the final workbook is opened by the user, Microsoft Office Excel will take care of displaying the formula according the applications language. Translation is taken care of by the application!

The following line of code writes the formula “=IF(C4>500,"profit","loss")” into the cell B8. Note that the formula must start with “=” to make PHPExcel recognise this as a formula.

$objPHPExcel->getActiveSheet()->setCellValue('B8','=IF(C4>500,"profit","loss")');

If you want to write a string beginning with an “=” to a cell, then you should use the setCellValueExplicit() method.

$objPHPExcel->getActiveSheet()

->setCellValueExplicit('B8',

'=IF(C4>500,"profit","loss")',

PHPExcel_Cell_DataType::TYPE_STRING

);

A cell’s formula can be read again using the following line of code:

$formula = $objPHPExcel->getActiveSheet()->getCell('B8')->getValue();

If you need the calculated value of a cell, use the following code. This is further explained in  REF _Ref191885372 \w \h  \* MERGEFORMAT 4.4.35.

$value = $objPHPExcel->getActiveSheet()->getCell('B8')->getCalculatedValue();

### Locale Settings for Formulae

Some localisation elements have been included in PHPExcel. You can set a locale by changing the settings. To set the locale to Russian you would use:

$locale = 'ru';

$validLocale = PHPExcel_Settings::setLocale($locale);

if (!$validLocale) {

echo 'Unable to set locale to '.$locale." - reverting to en_us<br />\n";

}

If Russian language files aren’t available, the setLocale() method will return an error, and English settings will be used throughout.

Once you have set a locale, you can translate a formula from its internal English coding.

$formula = $objPHPExcel->getActiveSheet()->getCell('B8')->getValue();

$translatedFormula =

PHPExcel_Calculation::getInstance()->_translateFormulaToLocale($formula);

You can also create a formula using the function names and argument separators appropriate to the defined locale; then translate it to English before setting the cell value:

$formula = '=ДНЕЙ360(ДАТА(2010;2;5);ДАТА(2010;12;31);ИСТИНА)';

$internalFormula =

PHPExcel_Calculation::getInstance()->translateFormulaToEnglish($formula);

$objPHPExcel->getActiveSheet()->setCellValue('B8',$internalFormula);

Currently, formula translation only translates the function names, the constants TRUE and FALSE, and the function argument separators.

At present, the following locale settings are supported:

__Language__

__Locale Code__

Czech

Čeština

cs

Danish

Dansk

da

German

Deutsch

de

Spanish

Español

es

Finnish

Suomi

fi

French

Français

fr

Hungarian

Magyar

hu

Italian

Italiano

it

Dutch

Nederlands

nl

Norwegian

Norsk

no

Polish

Język polski

pl

Portuguese

Português

pt

Brazilian Portuguese

Português Brasileiro

pt_br

Russian

русский язык

ru

Swedish

Svenska

sv

Turkish

Türkçe

tr

### Write a newline character "\n" in a cell (ALT+"Enter")

In Microsoft Office Excel you get a line break in a cell by hitting ALT+"Enter". When you do that, it automatically turns on "wrap text" for the cell.

Here is how to achieve this in PHPExcel:

$objPHPExcel->getActiveSheet()->getCell('A1')->setValue("hello\nworld");

$objPHPExcel->getActiveSheet()->getStyle('A1')->getAlignment()->setWrapText(true);

__Tip__

Read more about formatting cells using getStyle() elsewhere.__Tip__

AdvancedValuebinder.php automatically turns on "wrap text" for the cell when it sees a newline character in a string that you are inserting in a cell. Just like Microsoft Office Excel. Try this:require_once 'PHPExcel/Cell/AdvancedValueBinder.php';PHPExcel_Cell::setValueBinder( new PHPExcel_Cell_AdvancedValueBinder() );$objPHPExcel->getActiveSheet()->getCell('A1')->setValue("hello\nworld");Read more about AdvancedValueBinder.php elsewhere.### Explicitly set a cell’s datatype

You can set a cell’s datatype explicitly by using the cell’s setValueExplicit method, or the setCellValueExplicit method of a worksheet. Here’s an example:

$objPHPExcel->getActiveSheet()->getCell('A1')->setValueExplicit('25', PHPExcel_Cell_DataType::TYPE_NUMERIC);

### Change a cell into a clickable URL

You can make a cell a clickable URL by setting its hyperlink property:

$objPHPExcel->getActiveSheet()->setCellValue('E26', 'www.phpexcel.net');

$objPHPExcel->getActiveSheet()->getCell('E26')->getHyperlink()->setUrl('http://www.phpexcel.net');

If you want to make a hyperlink to another worksheet/cell, use the following code:

$objPHPExcel->getActiveSheet()->setCellValue('E26', 'www.phpexcel.net');

$objPHPExcel->getActiveSheet()->getCell('E26')->getHyperlink()->setUrl(“sheet://'Sheetname'!A1”);

### Setting a worksheet’s page orientation and size

Setting a worksheet’s page orientation and size can be done using the following lines of code:

$objPHPExcel->getActiveSheet()->getPageSetup()->setOrientation(PHPExcel_Worksheet_PageSetup::ORIENTATION_LANDSCAPE);

$objPHPExcel->getActiveSheet()->getPageSetup()->setPaperSize(PHPExcel_Worksheet_PageSetup::PAPERSIZE_A4);

Note that there are additional page settings available. Please refer to the API documentation for all possible options.

### Page Setup: Scaling options

The page setup scaling options in PHPExcel relate directly to the scaling options in the "Page Setup" dialog as shown in the illustration.

Default values in PHPExcel correspond to default values in MS Office Excel as shown in illustration



method

initial value

calling method will trigger

Note

setFitToPage(...)

false

-

setScale(...)

100

setFitToPage(false)

setFitToWidth(...)

1

setFitToPage(true)

value 0 means do-not-fit-to-width

setFitToHeight(...)

1

setFitToPage(true)

value 0 means do-not-fit-to-height

#### Example

Here is how to __f____it to 1 page wide by infinite pages tall__:

$objPHPExcel->getActiveSheet()->getPageSetup()->setFitToWidth(1);

$objPHPExcel->getActiveSheet()->getPageSetup()->setFitToHeight(0);

As you can see, it is not necessary to call setFitToPage(true) since setFitToWidth(…) and setFitToHeight(…) triggers this.

If you use setFitToWidth() you should in general also specify setFitToHeight() explicitly like in the example. Be careful relying on the initial values. This is especially true if you are upgrading from PHPExcel 1.7.0 to 1.7.1 where the default values for fit-to-height and fit-to-width changed from 0 to 1.

### Page margins

To set page margins for a worksheet, use this code:

$objPHPExcel->getActiveSheet()->getPageMargins()->setTop(1);
$objPHPExcel->getActiveSheet()->getPageMargins()->setRight(0.75);
$objPHPExcel->getActiveSheet()->getPageMargins()->setLeft(0.75);

$objPHPExcel->getActiveSheet()->getPageMargins()->setBottom(1);

Note that the margin values are specified in inches.



### Center a page horizontally/vertically

To center a page horizontally/vertically, you can use the following code:

$objPHPExcel->getActiveSheet()->getPageSetup()->setHorizontalCentered(true);
$objPHPExcel->getActiveSheet()->getPageSetup()->setVerticalCentered(false);

### Setting the print header and footer of a worksheet

Setting a worksheet’s print header and footer can be done using the following lines of code:

$objPHPExcel->getActiveSheet()->getHeaderFooter()->setOddHeader('&C&HPlease treat this document as confidential!');

$objPHPExcel->getActiveSheet()->getHeaderFooter()->setOddFooter('&L&B' . $objPHPExcel->getProperties()->getTitle() . '&RPage &P of &N');

Substitution and formatting codes (starting with &) can be used inside headers and footers. There is no required order in which these codes must appear.

The first occurrence of the following codes turns the formatting ON, the second occurrence turns it OFF again:

StrikethroughSuperscriptSubscriptSuperscript and subscript cannot both be ON at same time. Whichever comes first wins and the other is ignored, while the first is ON.

The following codes are supported by Excel2007:

&L

Code for "left section" (there are three header / footer locations, "left", "center", and "right"). When two or more occurrences of this section marker exist, the contents from all markers are concatenated, in the order of appearance, and placed into the left section.

&P

Code for "current page #"

&N

Code for "total pages"

&font size

Code for "text font size", where font size is a font size in points.

&K

Code for "text font color"

RGB Color is specified as RRGGBBTheme Color is specifed as TTSNN where TT is the theme color Id, S is either "+" or "-" of the tint/shade value, NN is the tint/shade value.&S

Code for "text strikethrough" on / off

&X

Code for "text super script" on / off

&Y

Code for "text subscript" on / off

&C

Code for "center section". When two or more occurrences of this section marker exist, the contents from all markers are concatenated, in the order of appearance, and placed into the center section.

&D

Code for "date"

&T

Code for "time"

&G

Code for "picture as background"

Please make sure to add the image to the header/footer:

$objDrawing = new PHPExcel_Worksheet_HeaderFooterDrawing();

$objDrawing->setName('PHPExcel logo');

$objDrawing->setPath('./images/phpexcel_logo.gif');

$objDrawing->setHeight(36);

$objPHPExcel->getActiveSheet()->getHeaderFooter()->addImage($objDrawing, PHPExcel_Worksheet_HeaderFooter::IMAGE_HEADER_LEFT);

&U

Code for "text single underline"

&E

Code for "double underline"

&R

Code for "right section". When two or more occurrences of this section marker exist, the contents from all markers are concatenated, in the order of appearance, and placed into the right section.

&Z

Code for "this workbook's file path"

&F

Code for "this workbook's file name"

&A

Code for "sheet tab name"

&+

Code for add to page #

&-

Code for subtract from page #

&"font name,font type"

Code for "text font name" and "text font type", where font name and font type are strings specifying the name and type of the font, separated by a comma. When a hyphen appears in font name, it means "none specified". Both of font name and font type can be localized values.

&"-,Bold"

Code for "bold font style"

&B

Code for "bold font style"

&"-,Regular"

Code for "regular font style"

&"-,Italic"

Code for "italic font style"

&I

Code for "italic font style"

&"-,Bold Italic"

Code for "bold italic font style"

&O

Code for "outline style"

&H

Code for "shadow style"

__Tip__

The above table of codes may seem overwhelming first time you are trying to figure out how to write some header or footer. Luckily, there is an easier way. Let Microsoft Office Excel do the work for you.For example, create in Microsoft Office Excel an xlsx file where you insert the header and footer as desired using the programs own interface. Save file as test.xlsx. Now, take that file and read off the values using PHPExcel as follows:$objPHPexcel = PHPExcel_IOFactory::load('test.xlsx');$objWorksheet = $objPHPexcel->getActiveSheet();var_dump($objWorksheet->getHeaderFooter()->getOddFooter());var_dump($objWorksheet->getHeaderFooter()->getEvenFooter());var_dump($objWorksheet->getHeaderFooter()->getOddHeader());var_dump($objWorksheet->getHeaderFooter()->getEvenHeader());That reveals the codes for the even/odd header and footer. Experienced users may find it easier to rename test.xlsx to test.zip, unzip it, and inspect directly the contents of the relevant xl/worksheets/sheetX.xml to find the codes for header/footer.### Setting printing breaks on a row or column

To set a print break, use the following code, which sets a row break on row 10.

$objPHPExcel->getActiveSheet()->setBreak( 'A10' , PHPExcel_Worksheet::BREAK_ROW );

The following line of code sets a print break on column D:

$objPHPExcel->getActiveSheet()->setBreak( 'D10' , PHPExcel_Worksheet::BREAK_COLUMN );

### Show/hide gridlines when printing

To show/hide gridlines when printing, use the following code:

$objPHPExcel->getActiveSheet()->setShowGridlines(true);

### Setting rows/columns to repeat at top/left

PHPExcel can repeat specific rows/cells at top/left of a page. The following code is an example of how to repeat row 1 to 5 on each printed page of a specific worksheet:

$objPHPExcel->getActiveSheet()->getPageSetup()->setRowsToRepeatAtTopByStartAndEnd(1, 5);

### Specify printing area

To specify a worksheet’s printing area, use the following code:

$objPHPExcel->getActiveSheet()->getPageSetup()->setPrintArea('A1:E5');

There can also be multiple printing areas in a single worksheet:

$objPHPExcel->getActiveSheet()->getPageSetup()->setPrintArea('A1:E5,G4:M20');

### Formatting cells

A cell can be formatted with font, border, fill, … style information. For example, one can set the  foreground colour of a cell to red, aligned to the right, and the border to black and thick border style. Let’s do that on cell B2:

$objPHPExcel->getActiveSheet()->getStyle('B2')->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_RED);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getAlignment()->setHorizontal(PHPExcel_Style_Alignment::HORIZONTAL_RIGHT);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getBorders()->getTop()->setBorderStyle(PHPExcel_Style_Border::BORDER_THICK);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getBorders()->getBottom()->setBorderStyle(PHPExcel_Style_Border::BORDER_THICK);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getBorders()->getLeft()->setBorderStyle(PHPExcel_Style_Border::BORDER_THICK);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getBorders()->getRight()->setBorderStyle(PHPExcel_Style_Border::BORDER_THICK);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);

$objPHPExcel->getActiveSheet()->getStyle('B2')->getFill()->getStartColor()->setARGB('FFFF0000');

Starting with PHPExcel 1.7.0 getStyle() also accepts a cell range as a parameter. For example, you can set a red background color on a range of cells:

$objPHPExcel->getActiveSheet()->getStyle('B3:B7')->getFill()

->setFillType(PHPExcel_Style_Fill::FILL_SOLID)

->getStartColor()->setARGB('FFFF0000');

__Tip__
It is recommended to style many cells at once, using e.g. getStyle('A1:M500'), rather than styling the cells individually in a loop. This is much faster compared to looping through cells and styling them individually.

There is also an alternative manner to set styles. The following code sets a cell’s style to font bold, alignment right, top border thin and a gradient fill:

$styleArray = array(

'font' => array(

'bold' => true,

),

'alignment' => array(

'horizontal' => PHPExcel_Style_Alignment::HORIZONTAL_RIGHT,

),

'borders' => array(

'top' => array(

'style' => PHPExcel_Style_Border::BORDER_THIN,

),

),

'fill' => array(

'type' => PHPExcel_Style_Fill::FILL_GRADIENT_LINEAR,

'rotation' => 90,

'startcolor' => array(

'argb' => 'FFA0A0A0',

),

'endcolor' => array(

'argb' => 'FFFFFFFF',

),

),

);

$objPHPExcel->getActiveSheet()->getStyle('A3')->applyFromArray($styleArray);

Or with a range of cells:

$objPHPExcel->getActiveSheet()->getStyle('B3:B7')->applyFromArray($styleArray);

This alternative method using arrays should be faster in terms of execution whenever you are setting more than one style property. But the difference may barely be measurable unless you have many different styles in your workbook.

Prior to PHPExcel 1.7.0 duplicateStyleArray() was the recommended method for styling a cell range, but this method has now been deprecated since getStyle() has started to accept a cell range.

### Number formats

You often want to format numbers in Excel. For example you may want a thousands separator plus a fixed number of decimals after the decimal separator. Or perhaps you want some numbers to be zero-padded.

In Microsoft Office Excel you may be familiar with selecting a number format from the "Format Cells" dialog. Here there are some predefined number formats available including some for dates. The dialog is designed in a way so you don't have to interact with the underlying raw number format code unless you need a custom number format.

In PHPExcel, you can also apply various predefined number formats. Example:

$objPHPExcel->getActiveSheet()->getStyle('A1')->getNumberFormat()

->setFormatCode(PHPExcel_Style_NumberFormat::FORMAT_NUMBER_COMMA_SEPARATED1);

This will format a number e.g. 1587.2 so it shows up as 1,587.20 when you open the workbook in MS Office Excel. (Depending on settings for decimal and thousands separators in Microsoft Office Excel it may show up as 1.587,20)

You can achieve exactly the same as the above by using this:

$objPHPExcel->getActiveSheet()->getStyle('A1')->getNumberFormat()

->setFormatCode('#,##0.00');

In Microsoft Office Excel, as well as in PHPExcel, you will have to interact with raw number format codes whenever you need some special custom number format. Example:

$objPHPExcel->getActiveSheet()->getStyle('A1')->getNumberFormat()

->setFormatCode('[Blue][>=3000]$#,##0;[Red][<0]$#,##0;$#,##0');

Another example is when you want numbers zero-padded with leading zeros to a fixed length:

$objPHPExcel->getActiveSheet()->getCell('A1')->setValue(19);

$objPHPExcel->getActiveSheet()->getStyle('A1')->getNumberFormat()

->setFormatCode('0000'); // will show as 0019 in Excel

__Tip__
The rules for composing a number format code in Excel can be rather complicated. Sometimes you know how to create some number format in Microsoft Office Excel, but don't know what the underlying number format code looks like. How do you find it?

The readers shipped with PHPExcel come to the rescue. Load your template workbook using e.g. Excel2007 reader to reveal the number format code. Example how read a number format code for cell A1:

$objReader = PHPExcel_IOFactory::createReader('Excel2007');
$objPHPExcel = $objReader->load('template.xlsx');
var_dump($objPHPExcel->getActiveSheet()->getStyle('A1')->getNumberFormat()->getFormatCode());
Advanced users may find it faster to inspect the number format code directly by renaming template.xlsx to template.zip, unzipping, and looking for the relevant piece of XML code holding the number format code in *xl/styles.xml*.### Alignment and wrap text

Let’s set vertical alignment to the top for cells A1:D4

$objPHPExcel->getActiveSheet()->getStyle('A1:D4')

->getAlignment()->setVertical(PHPExcel_Style_Alignment::VERTICAL_TOP);

Here is how to achieve wrap text:

$objPHPExcel->getActiveSheet()->getStyle('A1:D4')

->getAlignment()->setWrapText(true);

### Setting the default style of a workbook

It is possible to set the default style of a workbook. Let’s set the default font to Arial size 8:

$objPHPExcel->getDefaultStyle()->getFont()->setName('Arial');
$objPHPExcel->getDefaultStyle()->getFont()->setSize(8);

### Styling cell borders

In PHPExcel it is easy to apply various borders on a rectangular selection. Here is how to apply a thick red border outline around cells B2:G8.

$styleArray = array(

'borders' => array(

'outline' => array(

'style' => PHPExcel_Style_Border::BORDER_THICK,

'color' => array('argb' => 'FFFF0000'),

),

),

);

$objWorksheet->getStyle('B2:G8')->applyFromArray($styleArray);

In Microsoft Office Excel, the above operation would correspond to selecting the cells B2:G8, launching the style dialog, choosing a thick red border, and clicking on the "Outline" border component.

Note that the border outline is applied to the rectangular selection B2:G8 as a whole, not on each cell individually.

You can achieve any border effect by using just the 5 basic borders and operating on a single cell at a time:

__Array key__

__Maps to property__

left

right

top

bottom

diagonal

getLeft()
getRight()
getTop()
getBottom()
getDiagonal()

Additional shortcut borders come in handy like in the example above. These are the shortcut borders available:

__Array key__

__Maps to property__

allborders
outline

inside
vertical

horizontal

getAllBorders()

getOutline()

getInside()

getVertical()

getHorizontal()

An overview of all border shortcuts can be seen in the following image:



If you simultaneously set e.g. allborders and vertical, then we have "overlapping" borders, and one of the components has to win over the other where there is border overlap. In PHPExcel, from weakest to strongest borders, the list is as follows: allborders, outline/inside, vertical/horizontal, left/right/top/bottom/diagonal.

This border hierarchy can be utilized to achieve various effects in an easy manner.

### Conditional formatting a cell

A cell can be formatted conditionally, based on a specific rule. For example, one can set the foreground colour of a cell to red if its value is below zero, and to green if its value is zero or more.

One can set a conditional style ruleset to a cell using the following code:

$objConditional1 = new PHPExcel_Style_Conditional();

$objConditional1->setConditionType(PHPExcel_Style_Conditional::CONDITION_CELLIS);

$objConditional1->setOperatorType(PHPExcel_Style_Conditional::OPERATOR_LESSTHAN);

$objConditional1->addCondition('0');

$objConditional1->getStyle()->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_RED);

$objConditional1->getStyle()->getFont()->setBold(true);

$objConditional2 = new PHPExcel_Style_Conditional();

$objConditional2->setConditionType(PHPExcel_Style_Conditional::CONDITION_CELLIS);

$objConditional2->setOperatorType(PHPExcel_Style_Conditional::OPERATOR_GREATERTHANOREQUAL);

$objConditional2->addCondition('0');

$objConditional2->getStyle()->getFont()->getColor()->setARGB(PHPExcel_Style_Color::COLOR_GREEN);

$objConditional2->getStyle()->getFont()->setBold(true);

$conditionalStyles = $objPHPExcel->getActiveSheet()->getStyle('B2')->getConditionalStyles();

array_push($conditionalStyles, $objConditional1);

array_push($conditionalStyles, $objConditional2);

$objPHPExcel->getActiveSheet()->getStyle('B2')->setConditionalStyles($conditionalStyles);

If you want to copy the ruleset to other cells, you can duplicate the style object:

$objPHPExcel->getActiveSheet()->duplicateStyle( $objPHPExcel->getActiveSheet()->getStyle('B2'), 'B3:B7' );

### Add a comment to a cell

To add a comment to a cell, use the following code. The example below adds a comment to cell E11:

$objPHPExcel->getActiveSheet()->getComment('E11')->setAuthor('PHPExcel');

$objCommentRichText = $objPHPExcel->getActiveSheet()->getComment('E11')->getText()->createTextRun('PHPExcel:');


$objCommentRichText->getFont()->setBold(true);


$objPHPExcel->getActiveSheet()->getComment('E11')->getText()->createTextRun("\r\n");


$objPHPExcel->getActiveSheet()->getComment('E11')->getText()->createTextRun('Total amount on the current invoice, excluding VAT.');



### Apply autofilter to a range of cells

To apply an autofilter to a range of cells, use the following code:

$objPHPExcel->getActiveSheet()->setAutoFilter('A1:C9');

__Make sure that you always include the complete filter range!__
Excel does support setting only the captionrow, but that's __not__ a best practice...

### Setting security on a spreadsheet

Excel offers 3 levels of “protection”: document security, sheet security and cell security.

Document security allows you to set a password on a complete spreadsheet, allowing changes to be made only when that password is entered.Worksheet security offers other security options: you can disallow inserting rows on a specific sheet, disallow sorting, …Cell security offers the option to lock/unlock a cell as well as show/hide the internal formulaAn example on setting document security:

$objPHPExcel->getSecurity()->setLockWindows(true);

$objPHPExcel->getSecurity()->setLockStructure(true);

$objPHPExcel->getSecurity()->setWorkbookPassword("PHPExcel");

An example on setting worksheet security:

$objPHPExcel->getActiveSheet()->getProtection()->setPassword('PHPExcel');

$objPHPExcel->getActiveSheet()->getProtection()->setSheet(true);

$objPHPExcel->getActiveSheet()->getProtection()->setSort(true);

$objPHPExcel->getActiveSheet()->getProtection()->setInsertRows(true);

$objPHPExcel->getActiveSheet()->getProtection()->setFormatCells(true);

An example on setting cell security:

$objPHPExcel->getActiveSheet()->getStyle('B1')->getProtection()->setLocked(

PHPExcel_Style_Protection::PROTECTION_UNPROTECTED

);

__Make sure you enable worksheet protection if you need any of the worksheet protection features!__ This can be done using the following code: $objPHPExcel->getActiveSheet()->getProtection()->setSheet(true);

### Setting data validation on a cell

Data validation is a powerful feature of Excel2007. It allows to specify an input filter on the data that can be inserted in a specific cell. This filter can be a range (i.e. value must be between 0 and 10), a list (i.e. value must be picked from a list), …

The following piece of code only allows numbers between 10 and 20 to be entered in cell B3:

$objValidation = $objPHPExcel->getActiveSheet()->getCell('B3')

->getDataValidation();

$objValidation->setType( PHPExcel_Cell_DataValidation::TYPE_WHOLE );

$objValidation->setErrorStyle( PHPExcel_Cell_DataValidation::STYLE_STOP );

$objValidation->setAllowBlank(true);

$objValidation->setShowInputMessage(true);

$objValidation->setShowErrorMessage(true);

$objValidation->setErrorTitle('Input error');

$objValidation->setError('Number is not allowed!');

$objValidation->setPromptTitle('Allowed input');

$objValidation->setPrompt('Only numbers between 10 and 20 are allowed.');

$objValidation->setFormula1(10);

$objValidation->setFormula2(20);

The following piece of code only allows an item picked from a list of data to be entered in cell B3:

$objValidation = $objPHPExcel->getActiveSheet()->getCell('B5')

->getDataValidation();

$objValidation->setType( PHPExcel_Cell_DataValidation::TYPE_LIST );

$objValidation->setErrorStyle( PHPExcel_Cell_DataValidation::STYLE_INFORMATION );

$objValidation->setAllowBlank(false);

$objValidation->setShowInputMessage(true);

$objValidation->setShowErrorMessage(true);

$objValidation->setShowDropDown(true);

$objValidation->setErrorTitle('Input error');

$objValidation->setError('Value is not in list.');

$objValidation->setPromptTitle('Pick from list');

$objValidation->setPrompt('Please pick a value from the drop-down list.');

$objValidation->setFormula1('"Item A,Item B,Item C"');

When using a data validation list like above, make sure you put the list between " and " and that you split the items with a comma (,).

It is important to remember that any string participating in an Excel formula is allowed to be maximum 255 characters (not bytes). This sets a limit on how many items you can have in the string "Item A,Item B,Item C". Therefore it is normally a better idea to type the item values directly in some cell range, say A1:A3, and instead use, say, $objValidation->setFormula1('Sheet!$A$1:$A$3');. Another benefit is that the item values themselves can contain the comma ‘,’ character itself.

If you need data validation on multiple cells, one can clone the ruleset:

$objPHPExcel->getActiveSheet()->getCell('B8')->setDataValidation(clone $objValidation);

### Setting a column’s width

A column’s width can be set using the following code:

$objPHPExcel->getActiveSheet()->getColumnDimension('D')->setWidth(12);

If you want PHPExcel to perform an automatic width calculation, use the following code. PHPExcel will approximate the column with to the width of the widest column value.

$objPHPExcel->getActiveSheet()->getColumnDimension('B')->setAutoSize(true);

The measure for column width in PHPExcel does __not__ correspond exactly to the measure you may be used to in Microsoft Office Excel. Column widths are difficult to deal with in Excel, and there are several measures for the column width.1) __Inner width in character units__ (e.g. 8.43 this is probably what you are familiar with in Excel)2) __Full width in pixels__ (e.g. 64 pixels)3) __Full width in character units__ (e.g. 9.140625, value -1 indicates unset width)__PHPExcel always operates with 3) "Full width in character units"__ which is in fact the only value that is stored in any Excel file, hence the most reliable measure. Unfortunately, __Microsoft ____Office ____Excel does not present you with this ____measure__. Instead measures 1) and 2) are computed by the application when the file is opened and these values are presented in various dialogues and tool tips.The character width unit is the width of a '0' (zero) glyph in the workbooks default font. Therefore column widths measured in character units in two different workbooks can only be compared if they have the same default workbook font.If you have some Excel file and need to know the column widths in measure 3), you can read the Excel file with PHPExcel and echo the retrieved values.### Show/hide a column

To set a worksheet’s column visibility, you can use the following code. The first line explicitly shows the column C, the second line hides column D.

$objPHPExcel->getActiveSheet()->getColumnDimension('C')->setVisible(true);

$objPHPExcel->getActiveSheet()->getColumnDimension('D')->setVisible(false);

### Group/outline a column

To group/outline a column, you can use the following code:

$objPHPExcel->getActiveSheet()->getColumnDimension('E')->setOutlineLevel(1);

You can also collapse the column. Note that you should also set the column invisible, otherwise the collapse will not be visible in Excel 2007.

$objPHPExcel->getActiveSheet()->getColumnDimension('E')->setCollapsed(true);

$objPHPExcel->getActiveSheet()->getColumnDimension('E')->setVisible(false);

Please refer to the part “group/outline a row” for a complete example on collapsing.

You can instruct PHPExcel to add a summary to the right (default), or to the left. The following code adds the summary to the left:

$objPHPExcel->getActiveSheet()->setShowSummaryRight(false);

### Setting a row’s height

A row’s height can be set using the following code:

$objPHPExcel->getActiveSheet()->getRowDimension('10')->setRowHeight(100);

### Show/hide a row

To set a worksheet’s row visibility, you can use the following code. The following example hides row number 10.

$objPHPExcel->getActiveSheet()->getRowDimension('10')->setVisible(false);

Note that if you apply active filters using an AutoFilter, then this will override any rows that you hide or unhide manually within that AutoFilter range if you save the file.

### Group/outline a row

To group/outline a row, you can use the following code:

$objPHPExcel->getActiveSheet()->getRowDimension('5')->setOutlineLevel(1);

You can also collapse the row. Note that you should also set the row invisible, otherwise the collapse will not be visible in Excel 2007.

$objPHPExcel->getActiveSheet()->getRowDimension('5')->setCollapsed(true);

$objPHPExcel->getActiveSheet()->getRowDimension('5')->setVisible(false);

Here’s an example which collapses rows 50 to 80:

for ($i = 51; $i <= 80; $i++) {

$objPHPExcel->getActiveSheet()->setCellValue('A' . $i, "FName $i");

$objPHPExcel->getActiveSheet()->setCellValue('B' . $i, "LName $i");

$objPHPExcel->getActiveSheet()->setCellValue('C' . $i, "PhoneNo $i");

$objPHPExcel->getActiveSheet()->setCellValue('D' . $i, "FaxNo $i");

$objPHPExcel->getActiveSheet()->setCellValue('E' . $i, true);

$objPHPExcel->getActiveSheet()->getRowDimension($i)->setOutlineLevel(1);

$objPHPExcel->getActiveSheet()->getRowDimension($i)->setVisible(false);

}

$objPHPExcel->getActiveSheet()->getRowDimension(81)->setCollapsed(true);

You can instruct PHPExcel to add a summary below the collapsible rows (default), or above. The following code adds the summary above:

$objPHPExcel->getActiveSheet()->setShowSummaryBelow(false);

### Merge/unmerge cells

If you have a big piece of data you want to display in a worksheet, you can merge two or more cells together, to become one cell. This can be done using the following code:

$objPHPExcel->getActiveSheet()->mergeCells('A18:E22');

Removing a merge can be done using the unmergeCells method:

$objPHPExcel->getActiveSheet()->unmergeCells('A18:E22');

### Inserting rows/columns

You can insert/remove rows/columns at a specific position. The following code inserts 2 new rows, right before row 7:

$objPHPExcel->getActiveSheet()->insertNewRowBefore(7, 2);

### Add a drawing to a worksheet

A drawing is always represented as a separate object, which can be added to a worksheet. Therefore, you must first instantiate a new PHPExcel_Worksheet_Drawing, and assign its properties a meaningful value:

$objDrawing = new PHPExcel_Worksheet_Drawing();

$objDrawing->setName('Logo');

$objDrawing->setDescription('Logo');

$objDrawing->setPath('./images/officelogo.jpg');

$objDrawing->setHeight(36);

To add the above drawing to the worksheet, use the following snippet of code. PHPExcel creates the link between the drawing and the worksheet:

$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

You can set numerous properties on a drawing, here are some examples:

$objDrawing->setName('Paid');

$objDrawing->setDescription('Paid');

$objDrawing->setPath('./images/paid.png');

$objDrawing->setCoordinates('B15');

$objDrawing->setOffsetX(110);

$objDrawing->setRotation(25);

$objDrawing->getShadow()->setVisible(true);

$objDrawing->getShadow()->setDirection(45);

You can also add images created using GD functions without needing to save them to disk first as In-Memory drawings.

//  Use GD to create an in-memory image

$gdImage = @imagecreatetruecolor(120, 20) or die('Cannot Initialize new GD image stream');

$textColor = imagecolorallocate($gdImage, 255, 255, 255);

imagestring($gdImage, 1, 5, 5,  'Created with PHPExcel', $textColor);

//  Add the In-Memory image to a worksheet

$objDrawing = new PHPExcel_Worksheet_MemoryDrawing();

$objDrawing->setName('In-Memory image 1');

$objDrawing->setDescription('In-Memory image 1');

$objDrawing->setCoordinates('A1');

$objDrawing->setImageResource($gdImage);

$objDrawing->setRenderingFunction(

PHPExcel_Worksheet_MemoryDrawing::RENDERING_JPEG

);

$objDrawing->setMimeType(PHPExcel_Worksheet_MemoryDrawing::MIMETYPE_DEFAULT);

$objDrawing->setHeight(36);

$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

### Reading Images from a worksheet

A commonly asked question is how to retrieve the images from a workbook that has been loaded, and save them as individual image files to disk.

The following code extracts images from the current active worksheet, and writes each as a separate file.

$i = 0;

foreach ($objPHPExcel->getActiveSheet()->getDrawingCollection() as $drawing) {

if ($drawing instanceof PHPExcel_Worksheet_MemoryDrawing) {

ob_start();

call_user_func(

$drawing->getRenderingFunction(),

$drawing->getImageResource()

);

$imageContents = ob_get_contents();

ob_end_clean();

switch ($drawing->getMimeType()) {

case PHPExcel_Worksheet_MemoryDrawing::MIMETYPE_PNG :

$extension = 'png'; break;

case PHPExcel_Worksheet_MemoryDrawing::MIMETYPE_GIF:

$extension = 'gif'; break;

case PHPExcel_Worksheet_MemoryDrawing::MIMETYPE_JPEG :

$extension = 'jpg'; break;

}

} else {

$zipReader = fopen($drawing->getPath(),'r');

$imageContents = '';

while (!feof($zipReader)) {

$imageContents .= fread($zipReader,1024);

}

fclose($zipReader);

$extension = $drawing->getExtension();

}

$myFileName = '00_Image_'.++$i.'.'.$extension;

file_put_contents($myFileName,$imageContents);

}

### Add rich text to a cell

Adding rich text to a cell can be done using PHPExcel_RichText instances. Here’s an example, which creates the following rich text string:

This invoice is *__payable within thirty days after the end of the month__* unless specified otherwise on the invoice.

$objRichText = new PHPExcel_RichText();

$objRichText->createText('This invoice is ');

$objPayable = $objRichText->createTextRun('payable within thirty days after the end of the month');

$objPayable->getFont()->setBold(true);

$objPayable->getFont()->setItalic(true);

$objPayable->getFont()->setColor( new PHPExcel_Style_Color( PHPExcel_Style_Color::COLOR_DARKGREEN ) );

$objRichText->createText(', unless specified otherwise on the invoice.');

$objPHPExcel->getActiveSheet()->getCell('A18')->setValue($objRichText);

### Define a named range

PHPExcel supports the definition of named ranges. These can be defined using the following code:

// Add some data

$objPHPExcel->setActiveSheetIndex(0);

$objPHPExcel->getActiveSheet()->setCellValue('A1', 'Firstname:');

$objPHPExcel->getActiveSheet()->setCellValue('A2', 'Lastname:');

$objPHPExcel->getActiveSheet()->setCellValue('B1', 'Maarten');

$objPHPExcel->getActiveSheet()->setCellValue('B2', 'Balliauw');

// Define named ranges

$objPHPExcel->addNamedRange( new PHPExcel_NamedRange('PersonFN', $objPHPExcel->getActiveSheet(), 'B1') );

$objPHPExcel->addNamedRange( new PHPExcel_NamedRange('PersonLN', $objPHPExcel->getActiveSheet(), 'B2') );

Optionally, a fourth parameter can be passed defining the named range local (i.e. only usable on the current worksheet). Named ranges are global by default.

### Redirect output to a client’s web browser

Sometimes, one really wants to output a file to a client’s browser, especially when creating spreadsheets on-the-fly. There are some easy steps that can be followed to do this:

Create your PHPExcel spreadsheetOutput HTTP headers for the type of document you wish to outputUse the PHPExcel_Writer_* of your choice, and save to “php://output”PHPExcel_Writer_Excel2007 uses temporary storage when writing to php://output. By default, temporary files are stored in the script’s working directory. When there is no access, it falls back to the operating system’s temporary files location.

__This may not be safe for unauthorized viewing!__ 
Depending on the configuration of your operating system, temporary storage can be read by anyone using the same temporary storage folder. When confidentiality of your document is needed, it is recommended not to use php://output.

#### HTTP headers

Example of a script redirecting an Excel 2007 file to the client's browser:

<?php

/* Here there will be some code where you create $objPHPExcel */

// redirect output to client browser

header('Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');

header('Content-Disposition: attachment;filename="myfile.xlsx"');

header('Cache-Control: max-age=0');

$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel2007');

$objWriter->save('php://output');

?>

Example of a script redirecting an Excel5 file to the client's browser:

<?php

/* Here there will be some code where you create $objPHPExcel */

// redirect output to client browser

header('Content-Type: application/vnd.ms-excel');

header('Content-Disposition: attachment;filename="myfile.xls"');

header('Cache-Control: max-age=0');

$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');

$objWriter->save('php://output');

?>

Caution:

Make sure not to include any echo statements or output any other contents than the Excel file. There should be no whitespace before the opening <?php tag and at most one line break after the closing ?> tag (which can also be omitted to avoid problems).Make sure that your script is saved without a BOM (Byte-order mark). (Because this counts as echoing output)Same things apply to all included filesFailing to follow the above guidelines may result in corrupt Excel files arriving at the client browser, and/or that headers cannot be set by PHP (resulting in warning messages).

### Setting the default column width

Default column width can be set using the following code:

$objPHPExcel->getActiveSheet()->getDefaultColumnDimension()->setWidth(12);

### Setting the default row height

Default row height can be set using the following code:

$objPHPExcel->getActiveSheet()->getDefaultRowDimension()->setRowHeight(15);

### Add a GD drawing to a worksheet

There might be a situation where you want to generate an in-memory image using GD and add it to a PHPExcel worksheet without first having to save this file to a temporary location.

Here’s an example which generates an image in memory and adds it to the active worksheet:

// Generate an image

$gdImage = @imagecreatetruecolor(120, 20) or die('Cannot Initialize new GD image stream');

$textColor = imagecolorallocate($gdImage, 255, 255, 255);

imagestring($gdImage, 1, 5, 5,  'Created with PHPExcel', $textColor);

// Add a drawing to the worksheet

$objDrawing = new PHPExcel_Worksheet_MemoryDrawing();

$objDrawing->setName('Sample image');

$objDrawing->setDescription('Sample image');

$objDrawing->setImageResource($gdImage);

$objDrawing->setRenderingFunction(PHPExcel_Worksheet_MemoryDrawing::RENDERING_JPEG);

$objDrawing->setMimeType(PHPExcel_Worksheet_MemoryDrawing::MIMETYPE_DEFAULT);

$objDrawing->setHeight(36);

$objDrawing->setWorksheet($objPHPExcel->getActiveSheet());

### Setting worksheet zoom level

To set a worksheet’s zoom level, the following code can be used:

$objPHPExcel->getActiveSheet()->getSheetView()->setZoomScale(75);

Note that zoom level should be in range 10 – 400.

### Sheet tab color

Sometimes you want to set a color for sheet tab. For example you can have a red sheet tab:

$objWorksheet->getTabColor()->setRGB('FF0000');

### Creating worksheets in a workbook

If you need to create more worksheets in the workbook, here is how:

$objWorksheet1 = $objPHPExcel->createSheet();

$objWorksheet1->setTitle('Another sheet');

Think of createSheet() as the "Insert sheet" button in Excel. When you hit that button a new sheet is appended to the existing collection of worksheets in the workbook.### Hidden worksheets (Sheet states)

Set a worksheet to be __hidden__ using this code:

$objPHPExcel->getActiveSheet()

->setSheetState(PHPExcel_Worksheet::SHEETSTATE_HIDDEN);

Sometimes you may even want the worksheet to be __“very hidden”__. The available sheet states are :

PHPExcel_Worksheet::SHEETSTATE_VISIBLE

PHPExcel_Worksheet::SHEETSTATE_HIDDEN

PHPExcel_Worksheet::SHEETSTATE_VERYHIDDEN

In Excel the sheet state “very hidden” can only be set programmatically, e.g. with Visual Basic Macro. It is not possible to make such a sheet visible via the user interface.### Right-to-left worksheet

Worksheets can be set individually whether column ‘A’ should start at left or right side. Default is left. Here is how to set columns from right-to-left.

// right-to-left worksheet

$objPHPExcel->getActiveSheet()

->setRightToLeft(true);

# Performing formula calculations

## Using the PHPExcel calculation engine

As PHPExcel represents an in-memory spreadsheet, it also offers formula calculation capabilities. A cell can be of a value type (containing a number or text), or a formula type (containing a formula which can be evaluated). For example, the formula "=SUM(A1:A10)" evaluates to the sum of values in A1, A2, ..., A10.

To calculate a formula, you can call the cell containing the formula’s method getCalculatedValue(), for example:

$objPHPExcel->getActiveSheet()->getCell('E11')->getCalculatedValue();

If you write the following line of code in the invoice demo included with PHPExcel, it evaluates to the value "64":



Another nice feature of PHPExcel's formula parser, is that it can automatically adjust a formula when inserting/removing rows/columns. Here's an example:



You see that the formula contained in cell E11 is "SUM(E4:E9)". Now, when I write the following line of code, two new product lines are added:

$objPHPExcel->getActiveSheet()->insertNewRowBefore(7, 2);



Did you notice? The formula in the former cell E11 (now E13, as I inserted 2 new rows), changed to "SUM(E4:E11)". Also, the inserted cells duplicate style information of the previous cell, just like Excel's behaviour. Note that you can both insert rows and columns.

## Known limitations

There are some known limitations to the PHPExcel calculation engine. Most of them are due to the fact that an Excel formula is converted into PHP code before being executed. This means that Excel formula calculation is subject to PHP’s language characteristics.

### Operator precedence

In Excel '+' wins over '&', just like '*' wins over '+' in ordinary algebra. The former rule is not what one finds using the calculation engine shipped with PHPExcel.

Reference for operator precedence in Excel:

Reference for operator precedence in PHP:

### Formulas involving numbers and text

Formulas involving numbers and text may produce unexpected results or even unreadable file contents. For example, the formula '=3+"Hello "' is expected to produce an error in Excel (#VALUE!). Due to the fact that PHP converts “Hello” to a numeric value (zero), the result of this formula is evaluated as 3 instead of evaluating as an error. This also causes the Excel document being generated as containing unreadable content.

Reference for this behaviour in PHP:

[http://be.php.net/manual/en/language.types.string.php#language.types.string.conversion][20]

# Reading and writing to file

As you already know from part  REF _Ref191885438 \w \h 3.3  REF _Ref191885438 \h Readers and writers, reading and writing to a persisted storage is not possible using the base PHPExcel classes. For this purpose, PHPExcel provides readers and writers, which are implementations of PHPExcel_Writer_IReader and PHPExcel_Writer_IWriter.

## PHPExcel_IOFactory

The PHPExcel API offers multiple methods to create a PHPExcel_Writer_IReader or PHPExcel_Writer_IWriter instance:

Direct creationVia PHPExcel_IOFactoryAll examples underneath demonstrate the direct creation method. Note that you can also use the PHPExcel_IOFactory class to do this.

### Creating PHPExcel_Reader_IReader using PHPExcel_IOFactory

There are 2 methods for reading in a file into PHPExcel: using automatic file type resolving or explicitly.

Automatic file type resolving checks the different PHPExcel_Reader_IReader distributed with PHPExcel. If one of them can load the specified file name, the file is loaded using that PHPExcel_Reader_IReader. Explicit mode requires you to specify which PHPExcel_Reader_IReader should be used.

You can create a PHPExcel_Reader_IReader instance using PHPExcel_IOFactory in automatic file type resolving mode using the following code sample:

$objPHPExcel = PHPExcel_IOFactory::load("05featuredemo.xlsx");

A typical use of this feature is when you need to read files uploaded by your users, and you don’t know whether they are uploading xls or xlsx files.

If you need to set some properties on the reader, (e.g. to only read data, see more about this later), then you may instead want to use this variant:

$objReader = PHPExcel_IOFactory::createReaderForFile("05featuredemo.xlsx");

$objReader->setReadDataOnly(true);

$objReader->load("05featuredemo.xlsx");

You can create a PHPExcel_Reader_IReader instance using PHPExcel_IOFactory in explicit mode using the following code sample:

$objReader = PHPExcel_IOFactory::createReader("Excel2007");
$objPHPExcel = $objReader->load("05featuredemo.xlsx");

Note that automatic type resolving mode is slightly slower than explicit mode.

### Creating PHPExcel_Writer_IWriter using PHPExcel_IOFactory

You can create a PHPExcel_Writer_Iwriter instance using PHPExcel_IOFactory:

$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, "Excel2007");
$objWriter->save("05featuredemo.xlsx");

## Excel 2007 (SpreadsheetML) file format

Excel2007 file format is the main file format of PHPExcel. It allows outputting the in-memory spreadsheet to a .xlsx file.

### PHPExcel_Reader_Excel2007

#### Reading a spreadsheet

You can read an .xlsx file using the following code:

$objReader = new PHPExcel_Reader_Excel2007();

$objPHPExcel = $objReader->load("05featuredemo.xlsx");

#### Read data only

You can set the option setReadDataOnly on the reader, to instruct the reader to ignore styling, data validation, … and just read cell data:

$objReader = new PHPExcel_Reader_Excel2007();

$objReader->setReadDataOnly(true);

$objPHPExcel = $objReader->load("05featuredemo.xlsx");

#### Read specific sheets only

You can set the option setLoadSheetsOnly on the reader, to instruct the reader to only load the sheets with a given name:

$objReader = new PHPExcel_Reader_Excel2007();

$objReader->setLoadSheetsOnly( array("Sheet 1", "My special sheet") );

$objPHPExcel = $objReader->load("05featuredemo.xlsx");

#### Read specific cells only

You can set the option setReadFilter on the reader, to instruct the reader to only load the cells which match a given rule. A read filter can be any class which implements PHPExcel_Reader_IReadFilter. By default, all cells are read using the PHPExcel_Reader_DefaultReadFilter.

The following code will only read row 1 and rows 20 – 30 of any sheet in the Excel file:

class MyReadFilter implements PHPExcel_Reader_IReadFilter

{

public function readCell($column, $row, $worksheetName = '') {

// Read title row and rows 20 - 30

if ($row == 1 || ($row >= 20 && $row <= 30)) {

return true;

}

return false;

}

}

$objReader = new PHPExcel_Reader_Excel2007();

$objReader->setReadFilter( new MyReadFilter() );
$objPHPExcel = $objReader->load("06largescale.xlsx");

### PHPExcel_Writer_Excel2007

#### Writing a spreadsheet

You can write an .xlsx file using the following code:

$objWriter = new PHPExcel_Writer_Excel2007($objPHPExcel);

$objWriter->save("05featuredemo.xlsx");

#### Formula pre-calculation

By default, this writer pre-calculates all formulas in the spreadsheet. This can be slow on large spreadsheets, and maybe even unwanted. You can however disable formula pre-calculation:

$objWriter = new PHPExcel_Writer_Excel2007($objPHPExcel);
$objWriter->setPreCalculateFormulas(false);

$objWriter->save("05featuredemo.xlsx");

#### Office 2003 compatibility pack

Because of a bug in the Office2003 compatibility pack, there can be some small issues when opening Excel2007 spreadsheets (mostly related to formula calculation). You can enable Office2003 compatibility with the following code:

$objWriter = new PHPExcel_Writer_Excel2007($objPHPExcel);
$objWriter->setOffice2003Compatibility(true);

$objWriter->save("05featuredemo.xlsx");

__Office2003 compatibility should only be used when needed__
Office2003 compatibility option should only be used when needed. This option disables several Office2007 file format options, resulting in a lower-featured Office2007 spreadsheet when this option is used.

## Excel 5 (BIFF) file format

Excel5 file format is the old Excel file format, implemented in PHPExcel to provide a uniform manner to create both .xlsx and .xls files. It is basically a modified version of [PEAR Spreadsheet_Excel_Writer][21], although it has been extended and has fewer limitations and more features than the old PEAR library. This can read all BIFF versions that use OLE2: BIFF5 (introduced with office 95) through BIFF8, but cannot read earlier versions.

Excel5 file format will not be developed any further, it just provides an additional file format for PHPExcel.

__Excel5 (BIFF) limitations__
Please note that BIFF file format has some limits regarding to styling cells and handling large spreadsheets via PHP.

### PHPExcel_Reader_Excel5

#### Reading a spreadsheet

You can read an .xls file using the following code:

$objReader = new PHPExcel_Reader_Excel5();

$objPHPExcel = $objReader->load("05featuredemo.xls");

#### Read data only

You can set the option setReadDataOnly on the reader, to instruct the reader to ignore styling, data validation, … and just read cell data:

$objReader = new PHPExcel_Reader_Excel5();

$objReader->setReadDataOnly(true);

$objPHPExcel = $objReader->load("05featuredemo.xls");

#### Read specific sheets only

You can set the option setLoadSheetsOnly on the reader, to instruct the reader to only load the sheets with a given name:

$objReader = new PHPExcel_Reader_Excel5();

$objReader->setLoadSheetsOnly( array("Sheet 1", "My special sheet") );

$objPHPExcel = $objReader->load("05featuredemo.xls");

#### Read specific cells only

You can set the option setReadFilter on the reader, to instruct the reader to only load the cells which match a given rule. A read filter can be any class which implements PHPExcel_Reader_IReadFilter. By default, all cells are read using the PHPExcel_Reader_DefaultReadFilter.

The following code will only read row 1 and rows 20 – 30 of any sheet in the Excel file:

class MyReadFilter implements PHPExcel_Reader_IReadFilter

{

public function readCell($column, $row, $worksheetName = '') {

// Read title row and rows 20 - 30

if ($row == 1 || ($row >= 20 && $row <= 30)) {

return true;

}

return false;

}

}

$objReader = new PHPExcel_Reader_Excel5();

$objReader->setReadFilter( new MyReadFilter() );
$objPHPExcel = $objReader->load("06largescale.xls");

### PHPExcel_Writer_Excel5

#### Writing a spreadsheet

You can write an .xls file using the following code:

$objWriter = new PHPExcel_Writer_Excel5($objPHPExcel);

$objWriter->save("05featuredemo.xls");

## Excel 2003 XML file format

Excel 2003 XML file format is a file format which can be used in older versions of Microsoft Excel.

__Excel 2003 XML limitations__
Please note that Excel 2003 XML format has some limits regarding to styling cells and handling large spreadsheets via PHP.

### PHPExcel_Reader_Excel2003XML

#### Reading a spreadsheet

You can read an .xml file using the following code:

$objReader = new PHPExcel_Reader_Excel2003XML();

$objPHPExcel = $objReader->load("05featuredemo.xml");

#### Read specific cells only

You can set the option setReadFilter on the reader, to instruct the reader to only load the cells which match a given rule. A read filter can be any class which implements PHPExcel_Reader_IReadFilter. By default, all cells are read using the PHPExcel_Reader_DefaultReadFilter.

The following code will only read row 1 and rows 20 – 30 of any sheet in the Excel file:

class MyReadFilter implements PHPExcel_Reader_IReadFilter

{

public function readCell($column, $row, $worksheetName = '') {

// Read title row and rows 20 - 30

if ($row == 1 || ($row >= 20 && $row <= 30)) {

return true;

}

return false;

}

}

$objReader = new PHPExcel_Reader_Excel2003XML();

$objReader->setReadFilter( new MyReadFilter() );
$objPHPExcel = $objReader->load("06largescale.xml");

## Symbolic LinK (SYLK)

Symbolic Link (SYLK) is a Microsoft file format typically used to exchange data between applications, specifically spreadsheets. SYLK files conventionally have a .slk suffix. Composed of only displayable ANSI characters, it can be easily created and processed by other applications, such as databases.

__SYLK limitations__
Please note that SYLK file format has some limits regarding to styling cells and handling large spreadsheets via PHP.

### PHPExcel_Reader_SYLK

#### Reading a spreadsheet

You can read an .slk file using the following code:

$objReader = new PHPExcel_Reader_SYLK();

$objPHPExcel = $objReader->load("05featuredemo.slk");

#### Read specific cells only

You can set the option setReadFilter on the reader, to instruct the reader to only load the cells which match a given rule. A read filter can be any class which implements PHPExcel_Reader_IReadFilter. By default, all cells are read using the PHPExcel_Reader_DefaultReadFilter.

The following code will only read row 1 and rows 20 – 30 of any sheet in the SYLK file:

class MyReadFilter implements PHPExcel_Reader_IReadFilter

{

public function readCell($column, $row, $worksheetName = '') {

// Read title row and rows 20 - 30

if ($row == 1 || ($row >= 20 && $row <= 30)) {

return true;

}

return false;

}

}

$objReader = new PHPExcel_Reader_SYLK();

$objReader->setReadFilter( new MyReadFilter() );
$objPHPExcel = $objReader->load("06largescale.slk");

## Open/Libre Office (.ods)

Open Office or Libre Office .ods files are the standard file format fopr Open Office or Libre Office Calc files.

### PHPExcel_Reader_OOCalc

#### Reading a spreadsheet

You can read an .ods file using the following code:

$objReader = new PHPExcel_Reader_OOCalc();

$objPHPExcel = $objReader->load("05featuredemo.ods");

#### Read specific cells only

You can set the option setReadFilter on the reader, to instruct the reader to only load the cells which match a given rule. A read filter can be any class which implements PHPExcel_Reader_IReadFilter. By default, all cells are read using the PHPExcel_Reader_DefaultReadFilter.

The following code will only read row 1 and rows 20 – 30 of any sheet in the Calc file:

class MyReadFilter implements PHPExcel_Reader_IReadFilter

{

public function readCell($column, $row, $worksheetName = '') {

// Read title row and rows 20 - 30

if ($row == 1 || ($row >= 20 && $row <= 30)) {

return true;

}

return false;

}

}

$objReader = new PHPExcel_Reader_OOcalc();

$objReader->setReadFilter( new MyReadFilter() );
$objPHPExcel = $objReader->load("06largescale.ods");

## CSV (Comma Separated Values)

CSV (Comma Separated Values) are often used as an import/export file format with other systems. PHPExcel allows reading and writing to CSV files.

__CSV limitations__
Please note that CSV file format has some limits regarding to styling cells, number formatting, …

### PHPExcel_Reader_CSV

#### Reading a CSV file

You can read a .csv file using the following code:

$objReader = new PHPExcel_Reader_CSV();

$objPHPExcel = $objReader->load("sample.csv");

#### Setting CSV options

Often, CSV files are not really “comma separated”, or use semicolon (;) as a separator. You can instruct PHPExcel_Reader_CSV some options before reading a CSV file.

Note that PHPExcel_Reader_CSV by default assumes that the loaded CSV file is UTF-8 encoded. If you are reading CSV files that were created in Microsoft Office Excel the correct input encoding may rather be Windows-1252 (CP1252). Always make sure that the input encoding is set appropriately.

$objReader = new PHPExcel_Reader_CSV();
$objReader->setInputEncoding('CP1252');

$objReader->setDelimiter(';');

$objReader->setEnclosure('');

$objReader->setLineEnding("\r\n");
$objReader->setSheetIndex(0);

$objPHPExcel = $objReader->load("sample.csv");

#### Read a specific worksheet

CSV files can only contain one worksheet. Therefore, you can specify which sheet to read from CSV:

$objReader->setSheetIndex(0);

#### Read into existing spreadsheet

When working with CSV files, it might occur that you want to import CSV data into an existing PHPExcel object. The following code loads a CSV file into an existing $objPHPExcel containing some sheets, and imports onto the 6th sheet:

$objReader = new PHPExcel_Reader_CSV();
$objReader->setDelimiter(';');

$objReader->setEnclosure('');

$objReader->setLineEnding("\r\n");
$objReader->setSheetIndex(5); 
$objReader->loadIntoExisting("05featuredemo.csv", $objPHPExcel);

### PHPExcel_Writer_CSV

#### Writing a CSV file

You can write a .csv file using the following code:

$objWriter = new PHPExcel_Writer_CSV($objPHPExcel);

$objWriter->save("05featuredemo.csv");

#### Setting CSV options

Often, CSV files are not really “comma separated”, or use semicolon (;) as a separator. You can instruct PHPExcel_Writer_CSV some options before writing a CSV file:

$objWriter = new PHPExcel_Writer_CSV($objPHPExcel);
$objWriter->setDelimiter(';');

$objWriter->setEnclosure('');

$objWriter->setLineEnding("\r\n");
$objWriter->setSheetIndex(0);

$objWriter->save("05featuredemo.csv");

#### Write a specific worksheet

CSV files can only contain one worksheet. Therefore, you can specify which sheet to write to CSV:

$objWriter->setSheetIndex(0);

#### Formula pre-calculation

By default, this writer pre-calculates all formulas in the spreadsheet. This can be slow on large spreadsheets, and maybe even unwanted. You can however disable formula pre-calculation:

$objWriter = new PHPExcel_Writer_CSV($objPHPExcel);
$objWriter->setPreCalculateFormulas(false);

$objWriter->save("05featuredemo.csv");

#### Writing UTF-8 CSV files

A CSV file can be marked as UTF-8 by writing a BOM file header. This can be enabled by using the following code:

$objWriter = new PHPExcel_Writer_CSV($objPHPExcel);
$objWriter->setUseBOM(true);

$objWriter->save("05featuredemo.csv");

#### Decimal and thousands separators

If the worksheet you are exporting contains numbers with decimal or thousands separators then you should think about what characters you want to use for those before doing the export.

By default PHPExcel looks up in the server’s locale settings to decide what characters to use. But to avoid problems it is recommended to set the characters explicitly as shown below.

English users will want to use this before doing the export:

require_once 'PHPExcel/Shared/String.php'

PHPExcel_Shared_String::setDecimalSeparator('.');

PHPExcel_Shared_String::setThousandsSeparator(',');

German users will want to use the opposite values.

require_once 'PHPExcel/Shared/String.php'

PHPExcel_Shared_String::setDecimalSeparator(',');

PHPExcel_Shared_String::setThousandsSeparator('.');

Note that the above code sets decimal and thousand separators as global options. This also affects how HTML and PDF is exported.

## HTML

PHPExcel allows you to read or write a spreadsheet as HTML format, for quick representation of the data in it to anyone who does not have a spreadsheet application on their PC, or loading files saved by other scripts that simply create HTML markup and give it a .xls file extension.

__HTML limitations__
Please note that HTML file format has some limits regarding to styling cells, number formatting, …

### PHPExcel_Reader_HTML

#### Reading a spreadsheet

You can read an .html or .htm file using the following code:

$objReader = new PHPExcel_Reader_HTML();

$objPHPExcel = $objReader->load("05featuredemo.html");

__HTML limitations__
Please note that HTML reader is still experimental and does not yet support merged cells or nested tables cleanly

### PHPExcel_Writer_HTML

Please note that PHPExcel_Writer_HTML only outputs the first worksheet by default.

#### Writing a spreadsheet

You can write a .htm file using the following code:

$objWriter = new PHPExcel_Writer_HTML($objPHPExcel);

$objWriter->save("05featuredemo.htm");

#### Write all worksheets

HTML files can contain one or more worksheets. If you want to write all sheets into a single HTML file, use the following code:

$objWriter->writeAllSheets();

#### Write a specific worksheet

HTML files can contain one or more worksheets. Therefore, you can specify which sheet to write to HTML:

$objWriter->setSheetIndex(0);

#### Setting the images root of the HTML file

There might be situations where you want to explicitly set the included images root. For example, one might want to see <img  style="position: relative; left: 0px; top: 0px; width: 140px; height: 78px;" src="*http://www.domain.com/*images/logo.jpg" border="0"> instead of <img  style="position: relative; left: 0px; top: 0px; width: 140px; height: 78px;" src="./images/logo.jpg" border="0">.

You can use the following code to achieve this result:

$objWriter->setImagesRoot('http://www.example.com');

#### Formula pre-calculation

By default, this writer pre-calculates all formulas in the spreadsheet. This can be slow on large spreadsheets, and maybe even unwanted. You can however disable formula pre-calculation:

$objWriter = new PHPExcel_Writer_HTML($objPHPExcel);
$objWriter->setPreCalculateFormulas(false);

$objWriter->save("05featuredemo.htm");

#### Embedding generated HTML in a web page

There might be a situation where you want to embed the generated HTML in an existing website. PHPExcel_Writer_HTML provides support to generate only specific parts of the HTML code, which allows you to use these parts in your website.

Supported methods:

generateHTMLHeader()generateStyles()generateSheetData()generateHTMLFooter()Here’s an example which retrieves all parts independently and merges them into a resulting HTML page:

<?php
$objWriter = new PHPExcel_Writer_HTML($objPHPExcel);
echo $objWriter->generateHTMLHeader();
?>

<style>

<!--

html {

font-family: Times New Roman;

font-size: 9pt;

background-color: white;

}

<?php
echo $objWriter->generateStyles(false); // do not write <style> and </style>
?>

-->

</style>

<?php
echo $objWriter->generateSheetData();
echo $objWriter->generateHTMLFooter();
?>

#### Writing UTF-8 HTML files

A HTML file can be marked as UTF-8 by writing a BOM file header. This can be enabled by using the following code:

$objWriter = new PHPExcel_Writer_HTML($objPHPExcel);
$objWriter->setUseBOM(true);

$objWriter->save("05featuredemo.htm");

#### Decimal and thousands separators

See section PHPExcel_Writer_CSV how to control the appearance of these.

## PDF

PHPExcel allows you to write a spreadsheet into PDF format, for fast distribution of represented data.

__PDF limitations__
Please note that PDF file format has some limits regarding to styling cells, number formatting, …

### PHPExcel_Writer_PDF

PHPExcel’s PDF Writer is a wrapper for a 3rd-Party PDF Rendering library such as tcPDF, mPDF or DomPDF. Prior to version 1.7.8 of PHPExcel, the tcPDF library was bundled with PHPExcel; but from version 1.7.8 this was removed. Instead, you must now install a PDF Rendering library yourself; but PHPExcel will work with a number of different libraries.

Currently, the following libraries are supported:

__Library__

__Version used for testing__

__Downloadable from__

__PHPExcel Internal Constant__

tcPDF

5.9

http://www.tcpdf.org/

PDF_RENDERER_TCPDF

mPDF

5.4

http://www.mpdf1.com/mpdf/

PDF_RENDERER_MPDF

domPDF

0.6.0 beta 3

http://code.google.com/p/dompdf/

PDF_RENDERER_DOMPDF

The different libraries have different strengths and weaknesses. Some generate better formatted output than others, some are faster or use less memory than others, while some generate smaller .pdf files. It is the developers choice which one they wish to use, appropriate to their own circumstances.

Before instantiating a Writer to generate PDF output, you will need to indicate which Rendering library you are using, and where it is located.

$rendererName = PHPExcel_Settings::PDF_RENDERER_MPDF;

$rendererLibrary = 'mPDF5.4';

$rendererLibraryPath = dirname(__FILE__).'/../../../libraries/PDF/' . $rendererLibrary;

if (!PHPExcel_Settings::setPdfRenderer(

$rendererName,

$rendererLibraryPath

)) {

die(

'Please set the $rendererName and $rendererLibraryPath values' .

PHP_EOL .

' as appropriate for your directory structure'

);

}

#### Writing a spreadsheet

Once you have identified the Renderer that you wish to use for PDF generation, you can write a .pdf file using the following code:

$objWriter = new PHPExcel_Writer_PDF($objPHPExcel);

$objWriter->save("05featuredemo.pdf");

Please note that PHPExcel_Writer_PDF only outputs the first worksheet by default.

#### Write all worksheets

PDF files can contain one or more worksheets. If you want to write all sheets into a single PDF file, use the following code:

$objWriter->writeAllSheets();

#### Write a specific worksheet

PDF files can contain one or more worksheets. Therefore, you can specify which sheet to write to PDF:

$objWriter->setSheetIndex(0);

#### Formula pre-calculation

By default, this writer pre-calculates all formulas in the spreadsheet. This can be slow on large spreadsheets, and maybe even unwanted. You can however disable formula pre-calculation:

$objWriter = new PHPExcel_Writer_PDF($objPHPExcel);
$objWriter->setPreCalculateFormulas(false);

$objWriter->save("05featuredemo.pdf");

#### Decimal and thousands separators

See section PHPExcel_Writer_CSV how to control the appearance of these.

## Generating Excel files from templates (read, modify, write)

Readers and writers are the tools that allow you to generate Excel files from templates. This requires less coding effort than generating the Excel file from scratch, especially if your template has many styles, page setup properties, headers etc.

Here is an example how to open a template file, fill in a couple of fields and save it again:

$objPHPexcel = PHPExcel_IOFactory::load('template.xlsx');

$objWorksheet = $objPHPexcel->getActiveSheet();

$objWorksheet->getCell('A1')->setValue('John');

$objWorksheet->getCell('A2')->setValue('Smith');

$objWriter = PHPExcel_IOFactory::createWriter($objPHPexcel, 'Excel5');

$objWriter->save('write.xls');

Notice that it is ok to load an xlsx file and generate an xls file.

# Credits

Please refer to the internet page [http://www.codeplex.com/PHPExcel/Wiki/View.aspx?title=Credits&referringTitle=Home][22] for up-to-date credits.

# Valid array keys for style applyFromArray()

The following table lists the valid array keys for PHPExcel_Style applyFromArray() classes. If the “Maps to property” column maps a key to a setter, the value provided for that key will be applied directly. If the “Maps to property” column maps a key to a getter, the value provided for that key will be applied as another style array.

__PHPExcel_Style__

Array key:

Maps to property:

fill

font

borders

alignment

numberformat

protection


getFill()
getFont()
getBorders()
getAlignment()
getNumberFormat()
getProtection()

__PHPExcel_Style_Fill__

Array key:

Maps to property:

type
rotation
startcolor
endcolor
color


setFillType()
setRotation()
getStartColor()
getEndColor()
getStartColor()

__PHPExcel_Style_Font__

Array key:

Maps to property:

name
bold
italic
underline
strike
color
size
superScript
subScript


setName()
setBold()
setItalic()
setUnderline()
setStrikethrough()
getColor()
setSize()
setSuperScript()
setSubScript()

__PHPExcel_Style_Borders__

Array key:

Maps to property:

allborders
left
right
top
bottom
diagonal
vertical
horizontal
diagonaldirection
outline


getLeft(); getRight(); getTop(); getBottom()
getLeft()
getRight()
getTop()
getBottom()

getDiagonal()
getVertical()
getHorizontal()
setDiagonalDirection()
setOutline()

__PHPExcel_Style_Border__

Array key:

Maps to property:

style
color


setBorderStyle()
getColor()

__PHPExcel_Style_Alignment__

Array key:

Maps to property:

horizontal
vertical
rotation
wrap
shrinkToFit
indent


setHorizontal()
setVertical()
setTextRotation()
setWrapText()
setShrinkToFit()
setIndent()

__PHPExcel_Style_NumberFormat__

Array key:

Maps to property:

code


setFormatCode()

__PHPExcel_Style_Protection__

Array key:

Maps to property:

locked
hidden


setLocked()
setHidden()


  [2]: http://www.codeplex.com/PHPExcel/Wiki/View.aspx?title=Documents&referringTitle=Home
  [3]: http://www.ecma-international.org/news/TC45_current_work/TC45_available_docs.htm
  [4]: http://openxmldeveloper.org/articles/1970.aspx
  [5]: http://www.microsoft.com/downloads/details.aspx?familyid=941b3470-3ae9-4aee-8f43-c6bb74cd1466&displaylang=en
  [6]: http://www.codeplex.com/PackageExplorer/
  [7]: http://www.codeplex.com/PHPExcel/Wiki/View.aspx?title=FAQ&referringTitle=Requirements
  [8]: http://snaps.php.net/win32/php5.2-win32-latest.zip
  [9]: http://phpexcel.codeplex.com/Thread/View.aspx?ThreadId=234150
  [10]: http://phpexcel.codeplex.com/Thread/View.aspx?ThreadId=242712
  [11]: http://http:/forum.joomla.org/viewtopic.php?f=304&t=433060
  [12]: http://www.yiiframework.com/wiki/101/how-to-use-phpexcel-external-library-with-yii/
  [13]: http://bakery.cakephp.org/articles/melgior/2010/01/26/simple-excel-spreadsheet-helper
  [14]: http://www.flynsarmy.com/2010/07/phpexcel-module-for-kohana-3/
  [15]: http://szpargalki.blogspot.com/2011/02/phpexcel-kohana-framework.html
  [16]: http://typo3.org/documentation/document-library/extension-manuals/phpexcel_library/1.1.1/view/toc/0/
  [17]: http://phpexcel.codeplex.com/discussions/211925
  [18]: http://g-ernaelsten.developpez.com/tutoriels/excel2007/
  [19]: http://journal.mycom.co.jp/articles/2009/03/06/phpexcel/index.html
  [20]: http://be.php.net/manual/en/language.types.string.php#language.types.string.conversion
  [21]: http://pear.php.net/package/Spreadsheet_Excel_Writer
  [22]: http://www.codeplex.com/PHPExcel/Wiki/View.aspx?title=Credits&referringTitle=Home
