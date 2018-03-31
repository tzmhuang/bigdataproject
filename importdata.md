The Data
================

### Interpreting the data

Copying from the user guide in `READ_ME.zip`:

The Single Family Loan-Level Dataset is split in calendar quarters, beginning with the first quarter of 1999 and ending with the quarter as of the Origination Cutoff Date. For each calendar quarter, there is

-   One "origination data" file containing loan-level origination information for all the loans originated during the quarter.

-   One "monthly performance data" file for all of the respective loans originated during the quarter.

Also:

*For each calendar quarter, there is one file containing loan origination data and one file containing monthly performance data for each loan in the origination data file. There are cases when the loan is in the origination file but not in the performance file. This would happen when the loan gets paid off in the month of origination or before first cycle begins. Refer to the File Layout and Data Dictionary section of this document for more information on what data is contained in each file.*

### Importing the data

After downloading the data, put the data in two fodlers

-   the *origination data* in one folder, called `origination` (these are the small files, `historical_data1_QnYYYY`)

-   the *monthly performance data* in another folder called `monthlyperformace` (these are the large files, `historical_data1_time_QnYYYY.txt`)

##### open the sparklyr connection

``` r
library(sparklyr)
library(dplyr)
sc <- spark_connect(master = "local", version = "2.1.0")
```

##### Import the origination data

(the `system.time()` just measures the time, it also executes the function, so you can remove it if you want. Just interesting to see how long it takes. This one took me about 30 seconds with only 3 years of data, and roughly 10 minutes of 300 % CPU usage with the monthly performance data....)

``` r
origclass <- c('integer','integer','character', 'integer', 'character', 'real', 'integer',
               'character','real','integer','integer','integer','real','character','character','character','character',
               'character','character','character','character', 'integer', 'integer','character','character' ,'character')


system.time(
  origin <- spark_read_csv(
  path = "path/to/origination/*.txt",
  sc = sc, name = "origin", delimiter = "|", header=FALSE,
  columns = origclass )
)
```

##### Import the monthly performance data

``` r
svcgclass <- c('character','integer','real','character', 'integer','integer','character','character',
               'character','integer','real','real','integer', 'integer', 'character','integer','integer',
               'integer','integer','integer','integer','real','real')

system.time(
  month <- spark_read_csv(
  path = "path/to/monthlyperformance/*.txt",
  sc = sc, name = "month", delimiter = "|", header=FALSE,
  columns = svcgclass)
)
```

##### Next, rename the columns

``` r
# origin names
names(origin) <- c('fico','dt_first_pi','flag_fthb','dt_matr','cd_msa',"mi_pct",'cnt_units',
                   'occpy_sts','cltv','dti','orig_upb','ltv','int_rt','channel','ppmt_pnlty',
                   'prod_type','st', 'prop_type','zipcode','id_loan','loan_purpose',
                   'orig_loan_term','cnt_borr','seller_name','servicer_name', 'flag_sc')


# monthly performance names
names(month) <- c('id_loan','svcg_cycle','current_upb','delq_sts','loan_age','mths_remng',
                  'repch_flag','flag_mod', 'cd_zero_bal',
                  'dt_zero_bal','current_int_rt','non_int_brng_upb','dt_lst_pi','mi_recoveries',
                  'net_sale_proceeds','non_mi_recoveries','expenses', 'legal_costs',
                  'maint_pres_costs','taxes_ins_costs','misc_costs','actual_loss', 'modcost')
```

Info about what each column in begins at p.7 in the User guide for origination data and p.12 for the monthly performance data.
