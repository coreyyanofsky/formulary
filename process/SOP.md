CCDD Standard operating procedure (SOP)
================
Nancy Zhu, Daniel Buijs
2020-07-06

### Introduction

The Canadian Clinical Drug Data Set (CCDD) is a national medicinal
product terminology owned and maintaiend by Health Canada in partnership
with Canada Health Infoway. It is used to support electronic prescribing
in Canada, filling the gap in drug terminology.

For a detailed description of the scope, please visit: [Canada Clinical
Drug Data Set
HomePage](https://infoscribe.infoway-inforoute.ca/display/CCDD/Canadian+Clinical+Drug+Data+Set)

### CCDD monthly generation SOP

The monthly CCDD generation process can be roughly divided into two
stages: QA generation and Generation. All codes used for generation and
the relevant documents are documented in a public Github account,
accessible at (<https://github.com/hres/formulary>)

This SOP will list processes performed at **the Data Science Unit in
RMOD at Health Canada**.

### Set up in RStudio

Clone formulary repository to RStudio

    git clone https://github.com/hres/formulary.git

### Pre QA generation:

Pre-QA generation is performed to capture any changes in DPD that would
impact previously generated concepts. It is usually conducted 4-5 days
before QA generation. Pre-QA generations are currently only provided in
English

#### Steps to run Pre-QA generation:

1.  Upload DPD extract files from Y Drive to RStudio Server, save under
    folder dpd\_{date of dpd extract}

use WinSCP to upload files: (To use winSCP, a ssh key need to be issued
to the user)

  - Log in to Rstudio server using SSH key

  - Navigate to the corresponding source and destination directory

  - Drag DPD extract file folder from source directory to directory on
    RStudio server

<!-- end list -->

2.  `git checkout sql-views` switch to sql-views branch in formulary
    repository

3.  update ccdd\_date in
    [ccdd-config.csv](https://github.com/hres/formulary/blob/sql-views/src/sql/test/ccdd-config.csv)
    file. ccdd\_date is the date on which generation would be performed.

4.  update DPD extract file path in \[dpdloader\]
    (<https://github.com/hres/formulary/tree/sql-views/src/sql/dpdloader>)
    `formulary/src/sql/dpdloader` update filepath in files
    \[dpdload.pgload\],
    \[dpdload\_ap.pgload\],\[dpdload\_dr.pgload\],\[dpdload\_ia.pgload\]
    (the filepath should match where DPD extracts are stored)

5.  update source definition files by merging with sql-views-french
    branch

<!-- end list -->

    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-mp-whitelist-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-mp-deprecations-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/TM_filter_master.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-tm-definitions-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-ntp-definitions-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-pseudodin-map-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-mp-brand-override-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-mp-deprecations-draft.csv
    git checkout sql-views-french ~/formulary/src/sql/test/ccdd-ntp-deprecations-draft.csv
    
    git commit -m 'merge changes in source files from sql-views-french branch'
    git push

6.  update hard-coded dates in
    [setup.sh](https://github.com/hres/formulary/blob/sql-views/src/sql/setup.sh)

update line 4

    ccdd_qa_release_date="20xxxxxx"

to match the latest entry in
<https://github.com/hres/formulary/tree/folder_reorg/QAfiles>.

update line 39

    db_previous_month='ccdd_20xx_xx_xx_xxxxxx'

to match the appropriate database (which can be found at
<https://shiny.hres.ca/adminer/> only within the VPN).

update line 5

    ccdd_current_release_date="20xxxxxx"

to match the date of `db_previous_month`.

7.  run script
    [setup.sh](https://github.com/hres/formulary/blob/sql-views/src/sql/setup.sh)
    in command line (Terminal)

<!-- end list -->

    cd ./src/sql
    PGHOST=rest.hc.local PGUSER={username of database} PGPASSWORD={password} ./setup.sh

8.  All files are generation in `./src/dist/{date of generation}`

9.  `git checkout folder_reorg` Switch to `folder_reorg` branch.

10. Create new folder with date as folder name in `~/Pre-check` and
    subfolder (This step can be done with user interface or at command
    line)

*In Command line*

    mkdir ~/formulary/Pre-check/{date of generation}
    mkdir ~/formulary/Pre-check/{date of generation}/{date of generation}_from_{previous release date}
    
    cp  ~/formulary/src/dist/{date of generation}/*full_release_[[:digit:]]*  ~/formulary/Pre-check/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/*release_candidate_[[:digit:]]*  ~/formulary/Pre-check/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/mp_ntp_tm_relationship_release_candidate_[[:digit:]]*  ~/formulary/Pre-check/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/coded_attribute*  ~/formulary/Pre-check/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/device-ntp*  ~/formulary/Pre-check/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/special_groupings_release*  ~/formulary/Pre-check/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/*[[:digit:]]_release_changes_*  ~/formulary/Pre-check/{date of generation}/{date of generation}_from_{previous generation date}

*In user interface*

  - On the right corner of RStudio Panel, navigate to
    `Home/formulary/Pre-check` folder

  - Click *New Folder*

  - A pop window appears, input the name of folder with date of
    generation

  - Using the interface, navigate to `Home/formulary/src/dist/`

  - Select the file you intend to move by clicking the empty box on the
    left side of the file

  - From the menu, click More \> Move…

  - From the pop up window, select the destination folder

<!-- end list -->

10. Commit and push the files to remote Github repository

<!-- end list -->

    git commit -m "Precheck files for CCDD"
    git push origin folder_reorg 

11. Check folder
    [Pre-check](https://github.com/hres/formulary/tree/folder_reorg/Pre-check)
    and email QA group

### QA generation

  - Input files:
      - DPD extract [Online DPD extract updated
        monthly](https://www.canada.ca/en/health-canada/services/drugs-health-products/drug-products/drug-product-database/what-data-extract-drug-product-database.html)
      - CCDD release in previous month
      - CCDD QA release in previous month
      - Combination Products\_master.csv
      - Units\_of\_Presentation\_master.csv
      - Ingredient\_Stem\_file\_master.csv
      - Special Grouping.xlsx [The above files can be found
        here](https://github.com/hres/formulary/tree/folder_reorg)
      - ccdd-config.csv (for date updates)
      - ntp\_dosage\_form\_map\_master.csv
      - ccdd-mp-brand-override-draft.csv
      - ccdd-tm-definitions-draft.csv
      - ccdd-ntp-definition-draft.csv
      - ccdd-pseudodin-map-draft.csv
      - ccdd-mp-whitelist-draft.csv
      - ccdd-ntp-deprecation-draft.csv
      - ccdd-mtp-deprecations-draft.csv
      - TM\_filter\_master.csv
      - ccdd-tm-groupings-draft.csv [The above files can be found
        here](https://github.com/hres/formulary/tree/sql-views-french/src)
  - Output files:
      - qa\_release files (ntp,tm,mp,mp\_ntp\_tm\_relationship,
        special\_groupings)
      - qa\_release\_changes files
      - qa\_duplicates files
      - DPD\_diff files

Scripts for the generation procedures are stored [in the sql-views
branch in the hres/formulary Github
account](https://github.com/hres/formulary/tree/sql-views-french/src/sql)

#### Steps to run QA generation:

1.Upload DPD extract files from Y Drive to RStudio Server, save under
folder dpd\_{date of dpd extract} (See step 1 in pre-check)

2.  `git checkout sql-views-french` switch to sql-views-french branch in
    formulary repository

3.  update ccdd\_date and dpd\_extract\_date in
    [ccdd-config.csv](https://github.com/hres/formulary/blob/sql-views/src/sql/test/ccdd-config.csv)
    file. ccdd\_date is the date on which QA generation are performed.
    dpd\_extract\_date is the date on which DPD online extract is
    updated each month.

4.  **update line 1,2 in
    [setup.sh](https://github.com/hres/formulary/blob/sql-views-french/src/sql/setup.sh)
    file.** Input *ccdd\_qa\_release\_date* and
    *ccdd\_current\_release\_date*, these are the dates from previous
    cycle of generation.

5.  **Update line 57 in setup.sh** with the name of database (from
    PostgreSQL) for previous month release candidate generation (This
    will set up the correct reference data for comparisons)

6.  update filepath for DPD extracts in
    [dpdload.pgload](https://github.com/hres/formulary/blob/sql-views/src/sql/dpdloader/dpdload.pgload),
    [dpdload\_ap.pgload](https://github.com/hres/formulary/blob/sql-views/src/sql/dpdloader/dpdload_ap.pgload),
    [dpdload\_dr.pgload](https://github.com/hres/formulary/blob/sql-views/src/sql/dpdloader/dpdload_dr.pgload),
    [dpdload\_ia.pgload](https://github.com/hres/formulary/blob/sql-views/src/sql/dpdloader/dpdload_ia.pgload)

local folder:`formulary/src/sql/dpdloader`

7.  run script
    [setup.sh](https://github.com/hres/formulary/blob/sql-views/src/sql/setup.sh)
    in command line (Terminal)

<!-- end list -->

    cd ./src/sql
    PGHOST=rest.hc.local PGUSER={username of database} PGPASSWORD={password} ./setup.sh qa

8.  All files are generation in `./src/dist/{date of generation}`

9.  `git checkout folder_reorg` Switch to `folder_reorg` branch.

10. Create new folder with date as folder name in `~/QAfiles` and
    subfolder (This step can be done with user interface or at command
    line)

<!-- end list -->

  - In Command line:

<!-- end list -->

    mkdir ~/formulary/QAfiles/{date of generation}
    mkdir ~/formulary/QAfiles/{date of generation}/{date of generation_from_{previous QA date}}
    mkdir ~/formulary/QAfiles/{date of generation}/DPD_diff
    
    cp  ~/formulary/src/dist/{date of generation}/*qa_release_[[:digit:]]*  ~/formulary/QAfiles/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/mp_ntp_tm_relationship_qa_release_fr_*  ~/formulary/QAfiles/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/special_groupings_qa_release*  ~/formulary/QAfiles/{date of generation}
    
    cp  ~/formulary/src/dist/{date of generation}/*qa_release_changes_*  ~/formulary/QAfiles/{date of generation}/{date of generation_from_{previous QA date}}
    cp  ~/formulary/src/dist/{date of generation}/*duplicates_name*  ~/formulary/QAfiles/{date of generation}/{date of generation_from_{previous QA date}}
    cp ~/formulary/src/dist/{date of generation}/*post_qa_relationship*  ~/formulary/QAfiles/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/coded_attribute*  ~/formulary/QAfiles/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/device-ntp*  ~/formulary/QAfiles/{date of generation}
    cp ~/formulary/src/dist/{date of generation}/*dpd*_changes.csv ~/formulary/QAfiles/{date of generation}/DPD_diff

11. Commit and push the files to remote Github repository

<!-- end list -->

    git commit -m "QA generation for CCDD"
    git push origin folder_reorg 

12. Check folder
    [QAfiles](https://github.com/hres/formulary/tree/folder_reorg/QAfiles)
    and email QA group

### Generation

Before running generation, changes from the QA team are incoporated in
the following manners:

  - Assign new tm code to some of the TM concepts in the
    TM\_filter\_master.csv

  - Add previously published MPs (products being made dormant or
    cancelled post market),to whitelist (ccdd-mp-whitelist-draft.csv),
    so those products will be returned to the CCDD generation as
    ‘Inactive’

  - Assign MP products which have the same MP formal name with
    descriptors (ccdd-mp-brand-override-draft.csv)

  - Therefore the following input files are updated or incorporated at
    the Generation step
    
      - ccdd-mp-whitelist-draft.csv
      - ccdd-tm-definitions-draft.csv
      - ccdd-ntp-definitions-draft.csv
      - ccdd-pseudodin-map-draft.csv
      - ccdd-mp-brand-override-draft.csv

  - Output files:
    
      - release candidate files (ntp,tm,mp,mp\_ntp\_tm\_relationship,
        special\_groupings)
      - release\_changes files
      - qa\_duplicates files

#### Steps to run generation:

1.  run script
    [write-new-concepts.sh](https://github.com/hres/formulary/blob/sql-views/src/sql/write-new-concepts.sh)
    to assign codes to new ntp, tm and pseudodin.

<!-- end list -->

    git checkout sql-views-french
    cd ./src/sql
    PGHOST=rest.hc.local PGUSER={username of database} PGPASSWORD={password} ./write-new-concepts.sh
    
    Press 'b' when prompt to continue

2.  save tm\_filter.csv and qa\_file.docx (sent in email from QA group)
    to `~/formulary/src/sql/test`

3.  Run script `~/formulary/src/sql/test/updates_for_Release.R` to
    update `ccdd-mp-whitelist-draft.csv`
    `ccdd-ntp-definitions-draft.csv` `ccdd-tm-definitions-draft.csv`
    `ccdd-pseudodin-map-draft.csv` (See instruction in script for
    execution)

4.  According to QA Release Data Changes report, update
    `ccdd-mp-deprecations-draft.csv` `ccdd-mp-brand-override-draft.csv`
    `ccdd-ntp-deprecations-draft.csv`

5.  `git commit -m 'Update definition files`

6.  `git push origin sql-views-french`

7.  Generate release
    files

<!-- end list -->

    PGHOST=rest.hc.local PGUSER={username of database} PGPASSWORD={password} ./setup.sh

6.  All files are generation in `./src/dist/{date of generation}`

7.  `git checkout folder_reorg` Switch to `folder_reorg` branch.

8.  Create new folder with date as folder name in `~/formulary/releases`
    and subfolder (This step can be done with user interface or at
    command line)

**Command Line:**

    mkdir ~/formulary/releases/{date of generation}
    mkdir ~/formulary/releases/{date of generation}/{date of generation_from_{previous release date}}
    
    cp  ~/formulary/src/dist/{date of generation}/*full_release_[[:digit:]]*  ~/formulary/releases/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/mp_ntp_tm_relationship_fr_*  ~/formulary/releases/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/mp_ntp_tm_relationship_release_candidate_[[:digit:]]*  ~/formulary/releases/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/coded_attribute*  ~/formulary/releases/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/device-ntp*  ~/formulary/releases/{date of generation}
    cp  ~/formulary/src/dist/{date of generation}/special_groupings_release*  ~/formulary/releases/{date of generation}
    
    
    cp  ~/formulary/src/dist/{date of generation}/*[[:digit:]]_release_changes_*  ~/formulary/releases/{date of generation}/{date of generation_from_{previous QA date}}

9.  Commit and push the files to remote Github repository

<!-- end list -->

    git commit -m "release generation for CCDD"
    git push origin folder_reorg 

10. Check folder
    [releases](https://github.com/hres/formulary/tree/folder_reorg/releases)
    and email QA group

All releases are saved in the folder
[releases](https://github.com/hres/formulary/tree/folder_reorg/releases)
with the filename including date of generation.

11. OSIP CCDD team confirms that the final set of tables are correct and
    inform InfoWay of the publications

### Troubleshooting:

This section aims to resolve some of the common errors encountered
during
generations.

#### 1\. ERROR: duplicate key value violates unique constraint “ccdd\_ingredient\_stem\_pk”DETAIL: Key already exists.

This error is caused by duplication in french translation of english
concepts. This is usually from inconsistency between
`ccdd-tm-definitions-draft.csv` and `Ingredient_Stem_file_master.csv`.
To resolve the error, double check if the Key concept is spelt
differently between these two files and correct them
correspondently.

#### 2.ERROR: duplicate key value violates unique constraint “ccdd\_dosage\_form\_pk”DETAIL: Key already exists.

This error can be caused by either inconsistency between
`ccdd-ntp-definitions-draft.csv` and `ntp_dosage_form_map_master.csv` OR
inconsistency within `ntp_dosage_form_map_master.csv`. It happened a
couple time where a single English concept is associated with more than
one French translation. To resolve the error, standardize one English
concept to ONLY one French translation.