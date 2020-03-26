# utl-find-id-associated-with-max-values-for-each-column
Find ids associated with max values for several variables
    Find ids associated with max values for several variables

    github
    https://tinyurl.com/sebnay7
    https://github.com/rogerjdeangelis/utl-find-id-associated-with-max-values-for-each-column

    I could not get a single proc means to handle more that one variable

    * this is all thta is needed to get the id of the max age;

    proc means data=have noprint max nway missing stackodsoutput;
       var age   ;
       output out=want_age(drop=_:) maxid(age(name)) = namMaxAge  max=maxAge;
    run;quit;

    Up to 40 obs WORK.WANT_AGE total obs=1

    Obs    NAMMAXAGE    MAXAGE

     1       Alice        13

    TWO SOLUTIONS

         a. Normalize
         b. Iterative proc means array and do over macro


    Stackoverflow
    https://tinyurl.com/vfgbnuw
    https://stackoverflow.com/questions/3249236/how-to-find-max-value-of-variable-for-each-unique-observation-in-stacked-datas

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data have;
     set sashelp.class;
     input name$ age height weight;
    cards4;
    Alice 13 56.5 84.0
    James 12 57.3 83.0
    Thomas 11 57.5 85.0
    John 12 59.0 99.5
    Jane 12 59.8 84.5
    ;;;;
    run;quit;
    *            _               _
      ___  _   _| |_ _ __  _   _| |_ ___
     / _ \| | | | __| '_ \| | | | __/ __|
    | (_) | |_| | |_| |_) | |_| | |_\__ \
     \___/ \__,_|\__| .__/ \__,_|\__|___/
                    |_|               _
     _ __   ___  _ __ _ __ ___   __ _| (_)_______
    | '_ \ / _ \| '__| '_ ` _ \ / _` | | |_  / _ \
    | | | | (_) | |  | | | | | | (_| | | |/ /  __/
    |_| |_|\___/|_|  |_| |_| |_|\__,_|_|_/___\___|

    ;

    WORK.WANTNRM total obs=3

      NAME     _NAME_    COL1

      Alice    AGE       13.0
      John     WEIGHT    99.5
      Jane     HEIGHT    59.8

    *
     _ __  _ __ ___   ___   _ __ ___   ___  __ _ _ __  ___
    | '_ \| '__/ _ \ / __| | '_ ` _ \ / _ \/ _` | '_ \/ __|
    | |_) | | | (_) | (__  | | | | | |  __/ (_| | | | \__ \
    | .__/|_|  \___/ \___| |_| |_| |_|\___|\__,_|_| |_|___/
    |_|
    ;

     WORK.WANT total obs=1

      MAX_AGE_              MAX_HEIGHT_                 MAX_WEIGHT_
        NAME      AGE          NAME        HEIGHT          NAME        WEIGHT

       Alice       13          Jane         59.8           John         99.5

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|                               _
     _ __   ___  _ __ _ __ ___   __ _| (_)_______
    | '_ \ / _ \| '__| '_ ` _ \ / _` | | |_  / _ \
    | | | | (_) | |  | | | | | | (_| | | |/ /  __/
    |_| |_|\___/|_|  |_| |_| |_|\__,_|_|_/___\___|

    ;

    proc transpose data=have out=havXpo;
      by name notsorted;
      var age--weight;
    run;quit;

    /*
    Up to 40 obs from HAVXPO total obs=15

    Obs     NAME     _NAME_    COL1

      1    Alice     AGE       13.0
      2    Alice     HEIGHT    56.5
      3    Alice     WEIGHT    84.0
      4    James     AGE       12.0
      5    James     HEIGHT    57.3
      6    James     WEIGHT    83.0
    ....
    */

    proc sql;
      create
         table wantNrm as
      select
         *
      from
         havXpo
      where
         col1 in (
             select max(col1) from havXpo group by _name_)
    ;quit;

     _ __  _ __ ___   ___   _ __ ___   ___  __ _ _ __  ___
    | '_ \| '__/ _ \ / __| | '_ ` _ \ / _ \/ _` | '_ \/ __|
    | |_) | | | (_) | (__  | | | | | |  __/ (_| | | | \__ \
    | .__/|_|  \___/ \___| |_| |_| |_|\___|\__,_|_| |_|___/
    |_|
    ;

    %array(vars,values=age height weight);

    %do_over(vars,phrase=%str(
    proc means data=class noprint max nway missing;
       var ?;
       output out=?(drop=_:) maxid(?(name)) = max_?_name max=;
    run;quit;)
    );

    data want;
      merge
        %do_over(vars,Phrase=?)
    ;run;quit;



