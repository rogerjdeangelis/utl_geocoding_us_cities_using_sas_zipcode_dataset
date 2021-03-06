# utl_geocoding_us_cities_using_sas_zipcode_dataset
Geocoding US cities using sas zipcode dataset
    Geocoding US cities using sas zipcode dataset

    SAS Forum: Proc geocode issues

    see
    https://goo.gl/wkovR9
    https://communities.sas.com/t5/SAS-Procedures/Proc-geocode-issues/m-p/416702
    
    Free Geocoding 
    
    Texas A@M                                                                          
    http://geoservices.tamu.edu/Services/Geocode/BatchProcess/OverviewHowData.aspx     
                                                                                   
    UCLA Web Geocoder                                                                 
    https://gis.ucla.edu/geocoder                                                      
                                                                                   
    github
    https://tinyurl.com/yd9c7l94
    https://github.com/rogerjdeangelis/utl_geocode_and_reverse_geocode_netherland_addresses_and_latitudes_longitudes

    INPUT
    =====
    SAS LAT and LON are from zipcode dataset

    SD1.HAVE total obs=5

         ADR

      Poland,NY 13431
      Coleman,FL 33521
      San Antonio,TX 78248
      Rialto,CA 92376
      Vina,CA 96092


    WORKING CODE
    ============

     for ( i in 1:5) {
       lonlat_sample <- rbind(lonlat_sample, as.numeric(geocode(have$ADR[i])));
     };

    OUTPUT
    ======

     Up to 40 obs from sd1.lonlat total obs=5

       ADR                       R_LON      R_LAT

       Poland,NY 13431          -75.061    43.2256
       Coleman,FL 33521         -82.060    28.8010
       San Antonio,TX 78248     -98.525    29.5929
       Rialto,CA 92376         -117.370    34.1064
       Vina,CA 96092           -122.054    39.9326

    Comparison R latitudes and longitudes to those listed in SAS zipcode table

         R_LON       SAS         R_LAT       SAS

        -75.061    -75.073      43.2256    43.2051
        -82.060    -82.070      28.8010    28.8002
        -98.525    -98.523      29.5929    29.5841
       -117.370   -117.376      34.1064    34.1170
       -122.054   -122.012      39.9326    39.9377

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
      set sashelp.zipcode(rename=(x=lon y=lat)
           where=(statecode in ("NY","CA","TX","FL") and uniform(5031)<.0006));
      adr=cats(city,',',catx(' ',statecode,put(zip,z5.)));
      keep lat lon adr;
    run;quit;


    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

      __ _  ___  ___   ___ ___   __| | ___
     / _` |/ _ \/ _ \ / __/ _ \ / _` |/ _ \
    | (_| |  __/ (_) | (_| (_) | (_| |  __/
     \__, |\___|\___/ \___\___/ \__,_|\___|
     |___/
    ;

    %utl_submit_wps64('
    options set=R_HOME "C:/Program Files/R/R-3.3.1";
    libname sd1 "d:/sd1";
    proc r;
    submit;
    library(haven);
    library("ggmap");
    have<-read_sas("d:/sd1/have.sas7bdat");
    have;
    lonlat_sample <- c(NA,NA);
    for ( i in 1:5) {
      lonlat_sample <- rbind(lonlat_sample, as.numeric(geocode(have$ADR[i])));
    };
    want<-as.data.frame(lonlat_sample)[2:6,];
    want$ADR <- have$ADR;
    colnames(want)<-c("R_lon","R_lat","adr");
    endsubmit;
    import r=want data=sd1.lonlat;
    run;quit;
    ');

    proc print data=sd1.lonlat;
    run;quit;

    Obs      R_LON      R_LAT     ADR

     1      -75.061    43.2256    Poland,NY 13431
     2      -82.060    28.8010    Coleman,FL 33521
     3      -98.525    29.5929    San Antonio,TX 78248
     4     -117.370    34.1064    Rialto,CA 92376
     5     -122.054    39.9326    Vina,CA 96092



