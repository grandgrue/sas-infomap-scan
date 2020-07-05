# sas-infomap-scan
A python script to scan SAS Enterprise Guide (SEG) project files and SAS code files for InformationMap statements. 
It stores all the found information maps, variables and the path of the source file in a csv and optionaly in an excel sheet.
These outputs can be used, to analyze which information maps are used and to predict the dependency of changes to them.

## Modify config.json to suit your needs

| Setting  | Description | Example |
| :------- | :---------- | :------ | 
| temp_path | A working directory to unzip SEG project.xml files and to store the output csv (and optionaly Excel) files. Important: Use forward slashes even under Windows and no trailing slash! | "C:/Temp/sas-infomap-scan" | 
| write_to_excel | If set to false not Excel output file is created | true | 
| path_list | A list of path names that are scanned for files with the extension ".egp" and ".sas". Important: Use forward slashes even under Windows and no trailing slash! | ["N:/Directory", "X:/Pathname with blanks"] | 
| path_ignore | A list of path segments. If any of those strings is found in the pathname, the directory is ignored | ["/archive/", "/old/"] |

## Execute sas-infomap-scan.py
This is the main program and will scan the directories based on the config.json.

### Selecting files
If sas-infomap-scan.py finds a file with the extension ".egp" (which is internally a zip file) it will unzip "project.xml" to the temporary directory (see config) and parse it.
Files with the extension ".sas" are parsed directly.

### Scanning code

If a SEG generated libname assignement "libname \_egimle sasioime" is found, all the variables within the next "keep" statement are stored.
The name of the information map is extracted from the path of "mappath" setting.

In the following example the stored information map name is "BVS Wegzug int." and the variables are: FilterJahr, StichtagDatJahr, AnzWezuWir, HerkunftSort and ExportVersionCd.

```sas
libname _egimle sasioime
	 mappath="/InformationMaps/Bevoelkerung/BVS Wegzug int."        /* <-- INFORMATION MAP NAME */
	 aggregate=yes
	 metacredentials=no
	 PRESERVE_MAP_NAMES=YES
	 %SetDisplayOrder;

data WORK.WEG (label='Ausgewählte Daten von BVS Wegzug int.');
 	sysecho "Extrahieren von Daten aus der Information Map";
 	length 
		FilterJahr 8
		...
		;
 	label 
		FilterJahr="Jahresfilter"  /* Jahresfilter */
		...
		;
 	
 	set _egimle."BVS Wegzug int."n 
		(keep=
 			FilterJahr                                      /* <-- INFORMATION MAP VARIABLES */
 			StichtagDatJahr
 			AnzWezuWir
 			HerkunftSort
 			ExportVersionCd 
 		 /* default EXPCOLUMNLEN is 32 */ 
 		 filter=((FilterJahr &gt;= 1) AND NOT (ExportVersionCd = "A")) 
 		 
 		 );
 	
run;

/* clear the libname when complete */
libname _egimle clear;
```

The output csv of this example would look like this:

| infomap_name    | variables_keep  | filename                     |
| :-------------- | :-------------- | :--------------------------- | 
| BVS Wegzug int. | FilterJahr      | N:\Directory\Any Project.egp |
| BVS Wegzug int. | StichtagDatJahr | N:\Directory\Any Project.egp |
| BVS Wegzug int. | AnzWezuWir      | N:\Directory\Any Project.egp |
| BVS Wegzug int. | HerkunftSort    | N:\Directory\Any Project.egp |
| BVS Wegzug int. | ExportVersionCd | N:\Directory\Any Project.egp |

Note that the filenames contain backslashes for easier further processing unter Windows.

