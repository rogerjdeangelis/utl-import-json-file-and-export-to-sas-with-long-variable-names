# utl-import-json-file-and-export-to-sas-with-long-variable-names
Import json file and export to sas with long variable names
   %let pgm=utl-import-json-file-and-export-to-sas-with-long-variable-names;

    SAS Forum: Import json file and export to sas with long variable names

    GitHub
    https://tinyurl.com/2p8aa398
    Importing json files is more flexible in R and Python, because of the list data structure.

    https://github.com/rogerjdeangelis/utl-import-json-file-and-export-to-sas-with-long-variable-names

    The solution below provides a V5 transport that supports up to 40 byte SAS variable names.
    Long variable names are in the Vr Transport meta data.

    SAS Forum
    https://tinyurl.com/5xcxsc9j
    https://communities.sas.com/t5/SAS-Programming/Return-a-input-string-as-a-JSON-table/m-p/788653

    Note you can search all my names using the search term  'json in:name'.
    This lists my repors with json in the name

    Related json repos
    https://github.com/rogerjdeangelis?tab=repositories&q=json+in%3Aname&type=&language=&sort=


    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    filename ft15f001 "c:/temp/jsnstr.json";
    parmcards4;
    {"OrderID1": "A0000123", "OrderDate1": "4/30/2010","OrderID2": "A0000124", "OrderDate2": ""}
    ;;;;
    run;quit;

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    NOTE THE LONG VARIABLE NAMES FROM R

    Up to 40 obs WORK.WANT_LONG_NAMES total obs=1 07JAN2022:12:27:30

                         Order                  Order
    Obs    OrderID1      Date1      OrderID2    Date2

     1     A0000123    4/30/2010    A0000124

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    proc datasets lib=work kill ;
    run;quit;

    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_r64('
    library(jsonlite);
    library(data.table);
    library(tidyverse);
    library(SASxport);
    library(Hmisc);
    want <- as.data.frame(fromJSON("c:/temp/jsnstr.json"));
    for(i in seq_along(want)){
      Hmisc::label(want[, i]) <- colnames(want)[i]
    };
    want;
    write.xport(want,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";

    proc contents data=xpt.want;
    run;quit;

    * Note the label has long variable names;

    Alphabetic List of Variables and Attributes

    #    Variable    Type    Len    Label

    2    ORDERD      Char      9    OrderDate1
    4    ORDERD_1    Char      1    OrderDate1
    1    ORDERI      Char      8    OrderID1
    3    ORDERI_1    Char      8    OrderID1

    * rename using labels;

    proc datasets lib=work nolist nodetails mt=data mt=view;
    delete want want_long_names;
    run;quit;

    data want_long_names;
      %utl_rens(xpt.want);  * create view 'want' with long variable names;
      set want;
    run;quit;

    libname xpt clear;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
