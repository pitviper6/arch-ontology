# SCILHS i2b2 PCORnet Common Data Model (CDM) 
## Version 2.1.2 - 9/2016
##Jeff Klann, PhD

# Included Documentation Files

**INSTALL.md:** This file, explaining the installation process

**SCILHS i2b2 CDM Mapping Documentation.pdf:** Document the mechanism for creating local mappings. 

# Tasks

## Installing the ontology 

1. The ontology is broken down into eight tables. These tables use the standard ontology table structure with an added PCORI_BASECODE column and PCORNET_SPECIMEN_SOURCE in the labs table.
**In the Scripts folder, run the create_*_metadata_tables.sql file for your database type.**

2. The ontology is supplied as a set of pipe-delimited files, named pcornet_*.txt. Strings are double-quoted and null values are empty. There are no linebreaks in the strings, and all characters are standard ASCII. 
**Make sure to download the text files by clicking on the .txt file in Github, then click on the 'RAW' tab above the file text. Last, right click the raw formatted file and save to your preferred directory. If you fail to do this and just save-as before going to the raw format, the data import will fail.**
** Import the ontology files into separate tables named according to the file names. ** (These are the tables created in step 1 above.)
Note that the ontology contains one non-standard column, PCORI_BASECODE, and the pcornet_lab table has another: PCORI_SPECIMEN_SOURCE

  * pcornet_demo.txt - Demographics table - import into pcornet_demo. If you used the Demographics Excel spreadsheet to define your mappings, you will not need to import this file.
    * Note that there are different versions of this for Oracle vs. SqlServer.

  * pcornet_diag.txt - Diagnosis table - Import into pcornet_diag

  * pcornet_enc.txt - Encounter table - Import into pcornet_enc

  * pcornet_enroll.txt - Enrollment table - Import into pcornet_enroll

  * pcornet_proc.txt - Procedures table - Import into pcornet_proc. 

    * Alternately, pcornet_proc_nocpt.txt is included in the public release. This does not have CPT codes because one must have a CPT license to use CPT. Please import this into pcornet_proc instead. Contact SCILHS if you have a license and would like the full version.

  * pcornet_vital.txt - Vitals table - Import into pcornet_vital

  * pcornet_lab.txt - Labs table - Import into pcornet_lab

  * pcornet_med.txt - Medications table - Import into pcornet_med

3. **Import the TABLE_ACCESS table, also included as a txt file.** You can replace the rows in your existing TABLE_ACCESS table.

    * You need to replace the old entries in your TABLE_ACCESS table in order to perform queries. We recommend you use the provided file to ensure compatibility with the SHRINE pass-through mappings and forthcoming SMART-i2b2 configuration. It is fine to keep other entries for non-SCILHS ontologies in your TABLE_ACCESS table.

4. **Optionally import the SCHEMES table, also included as a txt file.** You can replace the rows in your existing SCHEMES table.

    * The SCHEMES table is used for searching by code when building queries in the webclient and workbench. 

    * Note that you will need to change the c_keys in the SCHEMES table after you finish mapping.

5. In the Scripts directory, **run the ontology-utils-* script** for your database to get the stored procedure utilities.

6. ** NEW STEP **: The age-at-visit tree that has been added to the Encounters table in 2.1.1 must be configured to point to your database. Please run the following code, replacing [mydb] with the database name you are using. In Oracle, also remove the ".dbo". **Note that some configurations work when removing the database name entirely (i.e. 'replacing [mydb].dbo.' with '')*
 `update pcornet_enc set c_dimcode=replace(C_DIMCODE,’PCORI_Dev.dbo.','[mydb].dbo.') where c_dimcode like '((select birth%'`

Also, run the following code block in order to add these expected columns to the visit_dimension table:
SQL code to add the columns (in MSSQL format) is below:
```
ALTER TABLE [dbo].[VISIT_DIMENSION]
ADD [DRG] varchar(50) NULL,
[DISCHARGE_STATUS] varchar(25) NULL,
[DISCHARGE_DISPOSITION] varchar(25) NULL, [LOCATION_ZIP] varchar(25) NULL,
[ADMITTING_SOURCE] varchar(25) NULL,
[FACILITYID] varchar(25) NULL,
[PROVIDERID] varchar(25) NULL
```
OR use the following code block for Oracle:
```
ALTER TABLE PCORNET.VISIT_DIMENSION
ADD (DRG varchar(50) NULL,
DISCHARGE_STATUS varchar(25) NULL,
DISCHARGE_DISPOSITION varchar(25) NULL,
LOCATION_ZIP varchar(25) NULL,
ADMITTING_SOURCE varchar(25) NULL,
FACILITYID varchar(25) NULL,
PROVIDERID varchar(25) NULL);
/
```

7. **Import the updated QT_BREAKDOWN_PATH file.** This updated table is located in the SCILHS Ontology repository. Click the following link for a direct path to the MSSQL file:
https://github.com/SCILHS/scilhs-ontology/blob/master/Ontology/QT_BREAKDOWN_PATH_MSSQL.txt



8. **Refresh your concept and modifier dimensions.** Stored procedures to do this are included in the refresh_dimensions script. All existing data is deleted and the tables are updated based on active tables in TABLE_ACCESS. You will need to modify these scripts if you use a separate database for the fact vs. ontology tables. Note also that the modifier dimension update script for Oracle is not finished. After running the script, execute the stored procedures. For example, in SqlServer, run:

```
exec dbo.FixConceptDim
go

exec dbo.FixModifierDim
go
```

At the end you might want to recompute stats on your database, especially if you are running Oracle:
```
BEGIN
DBMS_STATS.GATHER_SCHEMA_STATS(‘<schema_name>’, DBMS_STATS.AUTO_SAMPLE_SIZE);
END;
```
