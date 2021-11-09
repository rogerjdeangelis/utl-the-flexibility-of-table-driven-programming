# utl-the-flexibility-of-table-driven-programming
The flexibility of table driven programming  
    The flexibility of table driven programming                                                                                         
                                                                                                                                        
    GitHub                                                                                                                              
    https://tinyurl.com/hx7bdzar                                                                                                        
    https://github.com/rogerjdeangelis/utl-the-flexibility-of-table-driven-programming                                                  
                                                                                                                                        
    StackOverflow                                                                                                                       
    https://tinyurl.com/2krsj9au                                                                                                        
    https://stackoverflow.com/questions/69847619/how-can-i-store-a-macro-variable-value-into-sas-dataset                                
                                                                                                                                        
    It is worth looking at the other solutions which take advantage of the 'resolve' function and symget.                               
                                                                                                                                        
    It is often better to store the meta data and business rules in a separate object.                                                  
    Sometimes this is called table driven programming.                                                                                  
                                                                                                                                        
    You never touch the program, just edit the meta data.                                                                               
                                                                                                                                        
    Problem                                                                                                                             
                                                                                                                                        
       I have this meta data                                                                                                            
                                                                                                                                        
          WORK.META total obs=3 09NOV2021:13:34:14                                                                                      
                                                                                                                                        
          Obs    VAR                  FUNVAL                    UNIT                                                                    
                                                                                                                                        
           1     INVOICE    PUT(SUM(INVOICE)/1000,COMMAX18.2)    K                                                                      
           2     MODEL      PUT(COUNT(MODEL),COMMAX12.)          UN                                                                     
           3     MSRP       PUT(SUM(MSRP)/1000000,COMMAX12.)     K                                                                      
                                                                                                                                        
       and I need to replace funVal with sums from sashelp.cars                                                                         
                                                                                                                                        
       for example                                                                                                                      
                                                                                                                                        
           PUT(SUM(INVOICE)/1000,COMMAX18.2)                                                                                            
                                                                                                                                        
       needs to be replaced by                                                                                                          
                                                                                                                                        
           select PUT(SUM(INVOICE)/1000,COMMAX18.2) as funVal from sashelp.cars                                                         
    /*                   _                                                                                                              
    (_)_ __  _ __  _   _| |_                                                                                                            
    | | `_ \| `_ \| | | | __|                                                                                                           
    | | | | | |_) | |_| | |_                                                                                                            
    |_|_| |_| .__/ \__,_|\__|                                                                                                           
            |_|                                                                                                                         
    */                                                                                                                                  
                                                                                                                                        
    data meta;                                                                                                                          
     informat var $8. funVal $36. unit $4.;                                                                                             
     input  var funVal  unit;                                                                                                           
    cards4;                                                                                                                             
    INVOICE PUT(SUM(INVOICE)/1000,COMMAX18.2)  K                                                                                        
    MODEL   PUT(COUNT(MODEL),COMMAX12.)        UN                                                                                       
    MSRP    PUT(SUM(MSRP)/1000000,COMMAX12.)   K                                                                                        
                                                                                                                                        
    ;;;;                                                                                                                                
    run;quit;                                                                                                                           
                                                                                                                                        
    /*                                                                                                                                  
    Up to 40 obs from META total obs=3 09NOV2021:14:47:04                                                                               
                                                                                                                                        
    Obs    VAR                     FUNVAL                  UNIT                                                                         
                                                                                                                                        
     1     INVOICE    PUT(SUM(INVOICE)/1000,COMMAX18.2)     K                                                                           
     2     MODEL      PUT(COUNT(MODEL),COMMAX12.)           UN                                                                          
     3     MSRP       PUT(SUM(MSRP)/1000000,COMMAX12.)      K                                                                           
    */                                                                                                                                  
                                                                                                                                        
    /*           _               _                                                                                                      
      ___  _   _| |_ _ __  _   _| |_                                                                                                    
     / _ \| | | | __| `_ \| | | | __|                                                                                                   
    | (_) | |_| | |_| |_) | |_| | |_                                                                                                    
     \___/ \__,_|\__| .__/ \__,_|\__|                                                                                                   
                    |_|                                                                                                                 
                                                                                                                                        
    Up to 40 obs WORK.WANT total obs=3 09NOV2021:14:42:23                                                                               
                                                                                                                                        
    Obs    VAR        FUNVAL       UNIT    RC         STATUS                                                                            
                                                                                                                                        
     1     INVOICE    12.846,29     K       0    SQL SUM SUCCEEDED                                                                      
     2     MODEL      428           UN      0    SQL SUM SUCCEEDED                                                                      
     3     MSRP       14            K       0    SQL SUM SUCCEEDED                                                                      
                                                                                                                                        
     _ __  _ __ ___   ___ ___  ___ ___                                                                                                  
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|                                                                                                 
    | |_) | | | (_) | (_|  __/\__ \__ \                                                                                                 
    | .__/|_|  \___/ \___\___||___/___/                                                                                                 
    |_|                                                                                                                                 
    */                                                                                                                                  
                                                                                                                                        
    data want;                                                                                                                          
                                                                                                                                        
       set meta;                                                                                                                        
                                                                                                                                        
       call symputx("funVal",funVal);                                                                                                   
                                                                                                                                        
       rc=dosubl('                                                                                                                      
         proc sql;                                                                                                                      
          select                                                                                                                        
             &funVal                                                                                                                    
          into                                                                                                                          
             :outVal trimmed                                                                                                            
          from                                                                                                                          
            sashelp.cars                                                                                                                
          ;quit;                                                                                                                        
          %let cc=&syserr;                                                                                                              
                                                                                                                                        
         ');                                                                                                                            
                                                                                                                                        
         if symgetn('cc')=0 then status= "SQL SUM SUCCEEDED";                                                                           
         else status= "SQL SUM FAILED";                                                                                                 
                                                                                                                                        
         funVal=Symget('outVal');                                                                                                       
                                                                                                                                        
    run;quit;                                                                                                                           
                                                                                                                                        
    /*              _                                                                                                                   
      ___ _ __   __| |                                                                                                                  
     / _ \ `_ \ / _` |                                                                                                                  
    |  __/ | | | (_| |                                                                                                                  
     \___|_| |_|\__,_|                                                                                                                  
                                                                                                                                        
    */                                                                                                                                  
                                                         
