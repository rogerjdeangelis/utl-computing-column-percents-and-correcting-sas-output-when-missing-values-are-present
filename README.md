# utl-computing-column-percents-and-correcting-sas-output-when-missing-values-are-present
Computing column percents and correcting sas output when missing values are present
    %let pgm=utl-computing-column-percents-and-correcting-sas-output-when-missing-values-are-present;

    Computing column percents and correcting sas output when missing values are present;

    In addtion I want a sas dataset that is congruent with the tabulate listing


    github
    https://tinyurl.com/72fuzdv5
    https://github.com/rogerjdeangelis/utl-computing-column-percents-and-correcting-sas-output-when-missing-values-are-present

    SAS Communities
    https://tinyurl.com/37vcs2f7
    https://communities.sas.com/t5/SAS-Programming/How-to-output-missing-categories-with-proc-tabulate/m-p/779944

    Solution by
    PeterClemmensen
    https://communities.sas.com/t5/user/viewprofilepage/user-id/31304

    Conversion of tabulate output to SAS dataset was originally posted on SAS-L by
      Bartosz Jablonski
      yabwon@gmail.com

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    Tabulate produces this output for column percents

    --------------------------------------------------------------------
    |                                        |    2010    |    2020    |
    |----------------------------------------+------------+------------|
    |1                                       |           .|       50.00|
    |----------------------------------------+------------+------------|
    |2                                       |      100.00|       50.00|
    --------------------------------------------------------------------

    Missing is not correct for 2010 group 1

    I want an output dataset like this

      WORK.WANT total obs=2 12NOV2021:07:43:26

       GROUP    _2010    _2020

         1         0       50     * has 0 which os correct
         2       100       50

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    Data have(drop=x);
    Do x = 1 to 6;
          year = 2010;
          group = 2;
          output;
    End;
    Do x = 1 to 6;
          year = 2020;
          group = ceil(x/3);
          output;
    End;
    Run;

    Proc tabulate data=have out=not_want(drop=_type_ _page_ _table_);
    Class year / missing;
    Class group / missing;
    Table group='',year=''*colpctn='';
    Run;

    PROBLEM

    --------------------------------------------------------------------
    |                                        |    2010    |    2020    |
    |----------------------------------------+------------+------------|
    |1                                       |           .|       50.00|  * missing should be 0
    |----------------------------------------+------------+------------|
    |2                                       |      100.00|       50.00|
    --------------------------------------------------------------------

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */


    title;
    %utl_odstab(setup);
    Proc tabulate data=have out=not_want(drop=_type_ _page_ _table_);
       Class year / missing;
       Class group / missing;
       Table group='',year=''*colpctn='' / printmiss box=group;
    run;quit;
    %utl_odstab(outdsn=want);

    * reset option;
    options FORMCHAR='|----|+|---+=|-/\<>*';

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    ----------------------------------------
    |GROUP       |    2010    |    2020    |
    |------------+------------+------------|
    |1           |        0.00|       50.00|
    |------------+------------+------------|
    |2           |      100.00|       50.00|
    ----------------------------------------

      WORK.WANT total obs=2 12NOV2021:07:43:26

       GROUP    _2010    _2020

         1         0       50     * has 0 which os correct
         2       100       50
    /*        _       _           _
     _ __ ___| | __ _| |_ ___  __| |
    | `__/ _ \ |/ _` | __/ _ \/ _` |
    | | |  __/ | (_| | ||  __/ (_| |
    |_|  \___|_|\__,_|\__\___|\__,_|

    */
    github
    https://tinyurl.com/ybf9qtjj
    https://github.com/rogerjdeangelis/utl-creating-a-crosstab-output-sas-dataset-with-just-one-proc

    SAS forum
    https://tinyurl.com/y9ofewoa
    https://communities.sas.com/t5/SAS-Procedures/How-can-i-return-the-output-datasets-from-Proc-Tabulate/m-p/667113

    github
    https://tinyurl.com/ruak6u3
    https://github.com/rogerjdeangelis/utl-conformal-ods-table-output-for-proc-tabulate-and-proc-freq

    github
    https://tinyurl.com/y7dxgedy
    https://github.com/rogerjdeangelis/utl-crosstab-output-tables-from-corresp-report-not-static-tabulate

    related repo
    github
    https://tinyurl.com/w7qxsyx
    https://github.com/rogerjdeangelis/utl-fixing-ods-output-bug-in-proc-freq-crosstab-creating-ods-crosstab-table

    other repos
    https://github.com/rogerjdeangelis?tab=repositories&q=utl_odsrpt+in%3Areadme&type=&language=

    macros
    https://tinyurl.com/y9nfugth
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories


    /*
     _ __ ___   __ _  ___ _ __ ___
    | `_ ` _ \ / _` |/ __| `__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    */
    %macro utl_odstab(outdsn,datarow=0);

       %if %qupcase(&outdsn)=SETUP %then %do;

            filename _tmp1_ clear;  * just in case;

            %utlfkil(%sysfunc(pathname(work))/_tmp1_.txt);

            filename _tmp1_ "%sysfunc(pathname(work))/_tmp1_.txt";

            %let _ps_= %sysfunc(getoption(ps));
            %let _fc_= %sysfunc(getoption(formchar));

            OPTIONS ls=max ps=32756  FORMCHAR='|'  nodate nocenter;

            title; footnote;

            proc printto print=_tmp1_;
            run;quit;

       %end;
       %else %do;

            /* %let outdsn=tst; %let datarow=3; */

            proc printto;
            run;quit;

            %utlfkil(%sysfunc(pathname(work))/_tmp2_.txt);

            *filename _tmp2_  "%sysfunc(pathname(work))/_tmp2_.txt";

            proc datasets lib=work nolist;  *just in case;
             delete &outdsn;
            run;quit;

            proc printto print="%sysfunc(pathname(work))/_tmp2_.txt";
            run;quit;

            data _null_;
              retain n 0;
              infile _tmp1_ length=l;
              input lyn $varying32756. l;
              lyn=compress(lyn);
              if countc(lyn,'|')>2;
              n=n+1;
              if n ge %eval(&datarow + 1) then do;
                 file print;
                 putlog lyn;
                 put lyn;
              end;
            run;quit;

            proc printto;
            run;quit;

            proc import
               datafile="%sysfunc(pathname(work))/_tmp2_.txt"
               dbms=dlm
               out=&outdsn(drop=var:)
               replace;
               delimiter='|';
               getnames=yes;
            run;quit;

            filename _tmp1_ clear;
            filename _tmp2_ clear;

            %utlfkil(%sysfunc(pathname(work))/_tmp1_.txt);
            %utlfkil(%sysfunc(pathname(work))/_tmp2_.txt);

       %end;

    %mend utl_odstab;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */








