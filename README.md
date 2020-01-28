![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Find Orphaned Data Files With SQL
**Post Date: January 5, 2015**





## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process


<p>Often times whenever you are performing a database migration you need to determine which data files are actually being used, and which are not. You can always run a query against the the sys.master_files, and compare the physical_name to the actual files you can see in the data file path. This of course can be daunting as you may have hundreds or thousands of data files to sift through.
The best solution in this case is to build a list of all the actual data files that exist in the actual folder within the OS. Maybe; put that list into a table, and then run a comparison between that table, and the files which exist within SQL Server that are currently associated with live databases.
The following logic does exactly that utilizing the extended stored procedure xp_dirtree. One thing to keep in mind is that it will show you ONLY the data files that are orphaned under the form of .MDF, .NDF, and .LDF.   One other thing to watch out for is the list will return the following System Resource DB files:
Leave these files in place.  These can be ignored.
mssqlsystemresource.ldf
mssqlsystemresource.mdf

If you have custom data file extensions, or no extensions at all, then this logic might not work for you so I encourage you to modify and post back for others to use.
Anyway; on with the code! Hope you find it useful.</p>  


## SQL-Logic
```SQL
use master;
set nocount on
 
-- set the appropriate data file path.
declare @data_file_path varchar(255) 
set @data_file_path = 'D:\Microsoft SQL 2005\MSSQL.1\MSSQL\Data'
 
-- create temporary table to capture file names from directory.
if object_id('tempdb..#folderandfileinfo') is not null
 drop table #folderandfileinfo
 
create table #folderandfileinfo
(
 cid int identity(1,1) primary key clustered
, subdirectory varchar(255)
, depth int
, isfile int
)
 
-- populate temporary table with file names using xp_dirtree
insert into #folderandfileinfo
(
 [subdirectory]
, [depth]
, [isfile]
)
exec master..xp_dirtree
 @data_file_path
, 1
, 1
 
-- compare files found in the OS data file location to files associated with actual live databases.
-- WARNING: this does NOT take into consideration data files that have been detached. if you have detached
-- data files check with your DBA first before you clean up old orphaned data files.
select
 'path location' = @data_file_path
, 'orphaned data files' = subdirectory
 
from
 #folderandfileinfo
where
 subdirectory like '%df' -- compares ONLY to .mdf, .ndf, and .ldf
 and subdirectory not in
 (
 select right(smf.physical_name, charindex('\', reverse('\' + smf.physical_name)) - 1) 
 from
 sys.master_files smf join sys.databases sd on smf.database_id = sd.database_id
 )
order by
 subdirectory asc

```

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

 
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

