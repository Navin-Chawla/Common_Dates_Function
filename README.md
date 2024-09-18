# Common_Dates_Function
Generates hundreds of Date Variables based on a single Input Date.

# Imagine a Live Production Environment:

    [+] There are ~200 Production jobs having different frequency of execution i.e. on a Daily, Weekly & Monthly basis; in a Unix environment.
    
    [+] Each Production job triggers a different Unix Wrapper Script.
    
    [+] And each Wrapper Script executes a series of Teradata BTEQ scripts using sh command.
    
    [+] And inside each of these BTEQ scripts, there are multiple Date variables to be resolved, which are represented as ${D_LOAD_DT}, ${D_PREV_DT}, ${M_PREV_LAST_DT}, ${W_PREV_MON_DT} etc. in different formats like YYYY-MM-DD, YYYYMMDD, YYYY_MM_DD, YYYYMM, MMDD, YYYY, MM, DD etc.
    
    [+] Depending upon the specific SQL query in any BTEQ script, a different representation of date variable is used. For example:
        
        > A SELECT query to pull Source data for today: ${D_LOAD_DT}
        
        > A SELECT query to pull Source data for previous day: ${D_PREV_DT}
        
        > A SELECT query to pull Source data for last date of previous month: ${M_PREV_LAST_DT}
        
        > A SELECT query to pull Source data for last Monday: ${W_PREV_MON_DT}
       
        > A SELECT query to pull Source data for last Sunday: ${W_PREV_SUN_DT}
    
    [+] As you notice, Daily jobs' scripts will always use ${D_} date variables, Weekly ones will use ${W_} and Monthly ones will use {M_}.
    
    [+] Hierarchy is maintained in date representation by each job's scripts which means that Daily jobs can use ${D_}, ${W_} & ${M_} variables, Weekly jobs can only use ${W_} & ${M_} variables, and Monthly jobs can only use ${M_} variables.
    
    [+] A Unix variable 'JOB_DATE=' is used in the Wrapper which indicates the job's run date:
    
        > By default it is kept blank which indicates that the respective Wrapper's BTEQ scripts should capture today's latest data.
        
        > If any Daily job was skipped on a certain day, that day's date can be hardcoded as JOB_DATE=2024-12-31
        
        > If any Weekly job was skipped on a certain Monday, that Monday's date can be hardcoded as JOB_DATE=2024-12-30
        
        > If any Weekly job was skipped on a certain Friday, that Friday's date can be hardcoded as JOB_DATE=2024-12-27
        
        > If any Monthly job was skipped in a certain month, that month's last date can be hardcoded as JOB_DATE=2024-11-30

# Sample Unix Wrapper Script for 'Daily' jobs:

    #!/bin/bash

    echo "<Job_Name> job started at: `date`"

    # Job date should be defined here in YYYY-MM-DD format. Default value is Blank.
    JOB_DATE=

    # Generate the Date variables here and export them so they can be used in child programs i.e. BTEQ scripts
    export D_LOAD_DT=
    export D_PREV_DT=
    export D_LAST_DT=
    export W_PREV_SUN_DT=
    export M_PREV_LAST_DT=
    export D_PREV_QTR_LAST_DT=
    export M_PREV_LAST_WORKING_DAY=

    # Start calling the BTEQ scripts step-by-step
    sh 01_First_BTEQ_Script.sh
    ERR=$?
    if [ $ERR -ne 0 ]; then exit(1)

    sh 02_Second_BTEQ_Script.sh
    ERR=$?
    if [ $ERR -ne 0 ]; then exit(1)

    sh 03_Third_BTEQ_Script.sh
    ERR=$?
    if [ $ERR -ne 0 ]; then exit(1)

    echo "<Job_Name> job ended at: `date`"

# Problem Statement:

    [+] Currently, every Daily/Weekly/Monthly Wrapper creates its own Date variables ${D_}, ${W_} & ${M_} in required format depending on the value of JOB_DATE variable.  
    
    [+] This is creating a lot of redundancy in date calculation across different jobs.
    
    [+] Manual errors happen all the time in writing the code logic.

    [+] Coding the logic for one variable may span across multiple lines thus increasing the overall complexity of Wrapper.

    [+] Unix date calculation can be more erroneous than Python which has dedicated libraries like datetime & dateutil for writing custom logic.
    
    [+] New resources face difficulty in understanding the purpose of date calculation and writing the code logic themselves.

# Resolution:

    [+] Remove the manual calculations from Wrapper.
    
    [+] Create a Common Python script.

    [+] Pass the JOB_DATE variable to the Python script and access it using sys.argv[].

    [+] Depending whether JOB_DATE is Blank or hard-coded, generate all the hundreds of date variables into a file with export commands.

    [+] Replace the manual calculations by just calling the newly generated file.

    [+] As the newly generated file already has export commands for each date variable, the successive BTEQ scripts can easily access any of them.
