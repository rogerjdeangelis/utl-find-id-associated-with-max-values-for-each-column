# utl-find-id-associated-with-max-values-for-each-column
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


    Five SOLUTIONS

         a. Normalize
         b. Iterative proc means array macro and do over macro
         c. Array
             Bartosz Jablonski
             yabwon@gmail.com
         d. Hash
             Yinglin (Max) Wu
             yinglinwu@gmail.com
         e. R

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
                    |_|
      __ _     _ __   ___  _ __ _ __ ___   __ _| (_)_______
     / _` |   | '_ \ / _ \| '__| '_ ` _ \ / _` | | |_  / _ \
    | (_| |_  | | | | (_) | |  | | | | | | (_| | | |/ /  __/
     \__,_(_) |_| |_|\___/|_|  |_| |_| |_|\__,_|_|_/___\___|
    ;

    WORK.WANTNRM total obs=3

      NAME     _NAME_    COL1

      Alice    AGE       13.0
      John     WEIGHT    99.5
      Jane     HEIGHT    59.8

    *_
    | |__     _ __  _ __ ___   ___   _ __ ___   ___  __ _ _ __  ___
    | '_ \   | '_ \| '__/ _ \ / __| | '_ ` _ \ / _ \/ _` | '_ \/ __|
    | |_) |  | |_) | | | (_) | (__  | | | | | |  __/ (_| | | | \__ \
    |_.__(_) | .__/|_|  \___/ \___| |_| |_| |_|\___|\__,_|_| |_|___/
             |_|
    ;

     WORK.WANT total obs=1

      MAX_AGE_              MAX_HEIGHT_                 MAX_WEIGHT_
        NAME      AGE          NAME        HEIGHT          NAME        WEIGHT

       Alice       13          Jane         59.8           John         99.5

    *
      ___      __ _ _ __ _ __ __ _ _   _
     / __|    / _` | '__| '__/ _` | | | |
    | (__ _  | (_| | |  | | | (_| | |_| |
     \___(_)  \__,_|_|  |_|  \__,_|\__, |
                                   |___/
    ;

      WORK.WANT total obs=3

      NAME     VNAME      VAL

      Alice    AGE       13.0
      Jane     HEIGHT    59.8
      John     WEIGHT    99.5

    *    _     _               _
      __| |   | |__   __ _ ___| |__
     / _` |   | '_ \ / _` / __| '_ \
    | (_| |_  | | | | (_| \__ \ | | |
     \__,_(_) |_| |_|\__,_|___/_| |_|

    ;

    WORK.WANT total obs=3

      NAME     LABEL     VALUE

      John     WEIGHT     99.5
      Alice    AGE        13.0
      Jane     HEIGHT     59.8

    *         ____
      ___    |  _ \
     / _ \   | |_) |
    |  __/_  |  _ <
     \___(_) |_| \_\

    ;

    WORK.WANT total obs=3

    Obs    NAME     V2         V1

     1     Alice    AGE       13.0
     2     Jane     HEIGHT    59.8
     3     John     WEIGHT    99.5

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|                               _
      __ _     _ __   ___  _ __ _ __ ___   __ _| (_)_______
     / _` |   | '_ \ / _ \| '__| '_ ` _ \ / _` | | |_  / _ \
    | (_| |_  | | | | (_) | |  | | | | | | (_| | | |/ /  __/
     \__,_(_) |_| |_|\___/|_|  |_| |_| |_|\__,_|_|_/___\___|
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

    *_
    | |__     _ __  _ __ ___   ___   _ __ ___   ___  __ _ _ __  ___
    | '_ \   | '_ \| '__/ _ \ / __| | '_ ` _ \ / _ \/ _` | '_ \/ __|
    | |_) |  | |_) | | | (_) | (__  | | | | | |  __/ (_| | | | \__ \
    |_.__(_) | .__/|_|  \___/ \___| |_| |_| |_|\___|\__,_|_| |_|___/
             |_|
    ;


    data have;
     input name $ age height weight;
    cards4;
    Alice 13 56.5 84.0
    James 12 57.3 83.0
    Thomas 11 57.5 85.0
    John 12 59.0 99.5
    Jane 12 59.8 84.5
    ;;;;
    run;quit;

    %array(vars,values=age height weight);

    %do_over(vars,phrase=%str(
    proc means data=have noprint max nway missing;
       var ?;
       output out=?(drop=_:) maxid(?(name)) = max_?_name max=;
    run;quit;)
    );

    data want;
      merge
        %do_over(vars,Phrase=?)
    ;run;quit;

    *
      ___      __ _ _ __ _ __ __ _ _   _
     / __|    / _` | '__| '__/ _` | | | |
    | (__ _  | (_| | |  | | | (_| | |_| |
     \___(_)  \__,_|_|  |_|  \__,_|\__, |
                                   |___/
    ;

    data have;
     input name $ age height weight;
    cards4;
    Alice 13 56.5 84.0
    James 12 57.3 83.0
    Thomas 11 57.5 85.0
    John 12 59.0 99.5
    Jane 12 59.8 84.5
    ;;;;
    run;quit;

    data want;
      set have end = eof;

      array _NAME_ age height weight;
      array MAX[3] _TEMPORARY_;
      array MAXNAME[3] $ 32 _TEMPORARY_;

      do over _NAME_;
        if _NAME_ > MAX{_I_} then
          do;
            MAX{_I_}     = _NAME_;
            MAXNAME{_I_} =  NAME;
          end;
      end;

      if EOF then
        do over _NAME_;
          VNAME = vname(_NAME_);
          NAME  = MAXNAME{_I_};
          VAL   = MAX{_I_};
          output;
        end;

      keep VNAME NAME VAL;
    run;
    proc print;
    run;

    *    _     _               _
      __| |   | |__   __ _ ___| |__
     / _` |   | '_ \ / _` / __| '_ \
    | (_| |_  | | | | (_| \__ \ | | |
     \__,_(_) |_| |_|\__,_|___/_| |_|

    ;

    data have;
     input name $ age height weight;
    cards4;
    Alice 13 56.5 84.0
    James 12 57.3 83.0
    Thomas 11 57.5 85.0
    John 12 59.0 99.5
    Jane 12 59.8 84.5
    ;;;;
    run;quit;

    data _null_;
      declare hash h();
      length label $ 6;
      h.definekey('label');
      h.definedata('name', 'label', 'value');
      h.definedone();
      call missing(label, value);
      array ahw{*} age height weight;
      do while (not last);
          set have end=last;
          curr_name=name;
          do i=1 to 3;
                label=vname(ahw[i]);
                call missing(value);
                rc=h.find();
                name=curr_name;
                if ahw[i]>value then do;
                                value=ahw[i];
                                h.replace();
                end;
          end;
      end;
      h.output(dataset: 'want');
      stop;
    run;


    *         ____
      ___    |  _ \
     / _ \   | |_) |
    |  __/_  |  _ <
     \___(_) |_| \_\

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     input name $ age height weight;
    cards4;
    Alice 13 56.5 84.0
    James 12 57.3 83.0
    Thomas 11 57.5 85.0
    John 12 59.0 99.5
    Jane 12 59.8 84.5
    ;;;;
    run;quit;

    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_r64('
    library(data.table);
    library(haven);
    library(SASxport);
    have<-read_sas("d:/sd1/have.sas7bdat");
    idx<-apply(have[,-1],2, function(x) which(x == max(x), arr.ind=TRUE));
    maxes<-as.data.table(apply(have[,-1],2,max));
    want<-cbind(have[idx,1],names(idx),maxes);
    write.xport(want,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";
    data want;
      set xpt.want;
    run;quit;
    libname xpt clear;

