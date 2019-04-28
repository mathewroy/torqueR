torqueR
================
Mathew Roy  
April 27, 2019

What is torqueR?
----------------

torqueR is a demonstration of how to visualize your own vehicle's data using R.  
![Process Flow](/images/readme_flow.png)

Why did I create this?
----------------------

I wanted to understand what was going on with my car after it threw a check-engine light. My car was symptompatic and the diagnostic codes were related to the Mass Air Flow sensor (MAF), but this was only one plausibility. To find out more about the conditions leading to the symptoms, I logged my car's internal data for further study. The issue stemmed from a MAF that required some cleaning, and faulty replacement MAFs that I bought from a local auto parts store. While using torque wasn't directly involved in realizing that the MAF had to be cleaned, it was a great learning experience. My car's all good now.

How does it work?
-----------------

All modern cars are equipped with on-board diagnostics systems (OBD). These OBD systems provide owners and mechanics with access to information from the car's internal computers. Since the mid-90's all new cars are equipped with the latest OBD standard, called OBD-II, which provide us with various information on the car's inner workings.

Among other things, mechanics rely on information from the OBD-II system to diagnose check-engine lights and other issues. To analyze the car's data, an OBD-II scanner tool can be used to connect to the car's OBD-II port. This port is typically located below the steering column of the car. Nowadays, tools come in many forms, including Wireless OBD-II adapters which can plug-in to the OBD-II port and transmit information via Bluetooth or Wi-Fi to your Android Phone or other device.

I use a Bluetooth capable OBDII adapter to transmit information from my car. I receive this information using my Android phone and an app called Torque. The Torque app's settings are highly configurable - I have enabled logging so that any information collected are saved as time-stamped .csv files in a specific folder.

I use a second app called DriveSync (Drive as in Google Drive/OneDrive) to automatically upload any .csv files from my Torque App's log folder to my Google Drive.

Who will use torqueR
--------------------

These scripts are for anyone interested in playing around with their own car's data.

What is the goal of this project?
---------------------------------

The goal of this project is to show others how they can learn more about car data with OBDII, Torque, and R. It's a great way to use data for diagnostics and performance improvement.

Example: Visualize car data
===========================

Install & load packages
-----------------------

You need to install the following packages. Remove the hashtag to install.

``` r
# install.packages('googledrive')
# install.packages("readr")
# install.packages("dplyr")
# install.packages("ggplot2")
# install.packages("gganimate")
# install.packages("gifski")
# install.packages("ggpubr")

## Load required packages
lapply(c("googledrive","readr","dplyr","ggplot2","ggpubr","gganimate","gifski"), require, character.only = TRUE)
```

Import Torque data from your Google Drive
-----------------------------------------

If you have used the DriveSync app to auto-sync/upload your Torque app's logs to your Google Drive account (or if it's uploaded manually to Google Drive), you can import it using the following lines of code. The result will be a .CSV file that will be downloaded to your computer and then imported into R as a data frame.

If it's your first time using the googledrive package, you should allow for a cache of your credentials (enter '1' into the R Console window when asked). You will also need to give "tidyverse api packages" access to your Google Account (a new browser window will open, click "Allow").

torqueLogs is the folder in my Google Drive where my Torque app's logs are stored.

``` r
torquefiles <- googledrive::drive_ls("~/torqueLogs/")
head(torquefiles$name)
```

    ## [1] "trackLog-2018-Nov-06_16-56-54.csv" "trackLog-2018-Nov-06_16-56-43.csv"
    ## [3] "trackLog-2018-Nov-06_16-53-15.csv" "trackLog-2018-Nov-06_16-53-01.csv"
    ## [5] "trackLog-2018-Nov-06_16-40-23.csv" "trackLog-2018-Nov-06_16-40-12.csv"

Identify the name of the .csv file that you want to download. For this example, I already know that I am interested in the filename associated with index 117.

``` r
filename <- torquefiles$name[117]
drive_download(file = filename, overwrite = TRUE,verbose = FALSE)
print(paste0("Your CSV has been saved here: ",getwd()))
```

    ## [1] "Your CSV has been saved here: C:/Users/User/Documents/torqueR"

Import the downloaded torque data (.csv) into R as a data frame named 'trip'. We'll make a copy called 'trip2' to work with. The list of columns available for selection depends on what was enabled in the Torque App's log settings.

``` r
trip <- readr::read_csv(paste0(getwd(),"/",filename))
trip2 <- 
  trip %>% 
  # Select and rename variables of interest. 
  select(
     time="Device Time",
     runtime_s="Run time since engine start(s)",
     trpdist_k="Trip Distance(km)",
     trptime_s="Trip Time(Since journey start)(s)",
     trptimemov_s="Trip time(whilst moving)(s)",
     trptimestat_s="Trip time(whilst stationary)(s)",
     lat="Latitude",
     long="Longitude",
     latgps="GPS Latitude(°)",
     longgps="GPS Longitude(°)",
     
     rpm="Engine RPM(rpm)",
     speed="Speed (OBD)(km/h)",
     accsentot_g="Acceleration Sensor(Total)(g)",
     whp="Horsepower (At the wheels)(hp)",
     torque="Torque(ft-lb)",
     absthrpos_pc="Absolute Throttle Position B(%)",
     accpedposd_pc="Accelerator PedalPosition D(%)",                   
     accpedpose_pc="Accelerator PedalPosition E(%)",   
     
     ambairtem_c="Ambient air temp(°C)",
     intairtem_c="Intake Air Temperature(°C)",
     coolant_c="Engine Coolant Temperature(°C)", 
     catsenb1s1_c="Catalyst Temperature (Bank 1 Sensor 1)(°C)",
     
     barpres_psi="Barometric pressure (from vehicle)(psi)",
     intmanpres_psi="Intake Manifold Pressure(psi)",
     turvaccguage_psi="Turbo Boost & Vacuum Gauge(psi)",
     
     ftrimb1lt_pc="Fuel Trim Bank 1 Long Term(%)",
     ftrimb1st_pc="Fuel Trim Bank 1 Short Term(%)",
     ftrimb1s1_pc="Fuel trim bank 1 sensor 1(%)",
     o2b1s1_v="O2 Volts Bank 1 sensor 1(V)",
     o2b1s2_v="O2 Volts Bank 1 sensor 2(V)",
     
     afr="Air Fuel Ratio(Commanded)(:1)",
     lambda="Commanded Equivalence Ratio(lambda)",
     maf_gps="Mass Air Flow Rate(g/s)",
     fflow_lph="Fuel flow rate/hour(l/hr)",
     fflow_ccpmin="Fuel flow rate/minute(cc/min)",
     
     egrcom_pc="EGR Commanded(%)",
     egrerr_pc="EGR Error(%)",
     engload_pc="Engine Load(%)",
     engabsload_pc="Engine Load(Absolute)(%)",
     voleff_pc="Volumetric Efficiency (Calculated)(%)",
     
     kpl="Kilometers Per Litre(Instant)(kpl)",
     kpl_lt="Kilometers Per Litre(Long Term Average)(kpl)",
     lp100k="Litres Per 100 Kilometer(Instant)(l/100km)",
     lp100k_lt="Litres Per 100 Kilometer(Long Term Average)(l/100km)",
     
     frem_ecu_pc="Fuel Level (From Engine ECU)(%)",
     frem_app_pc="Fuel Remaining (Calculated from vehicle profile)(%)",
     trp_fuelused_l="Fuel used (trip)(l)",
     trp_lp100k="Trip average Litres/100 KM(l/100km)",
     
     controlmod_v="Voltage (Control Module)(V)",
     obd2_v="Voltage (OBD Adapter)(V)"
    ) %>% 
  mutate(
    # Convert time column into POSIXct type
    time = as.POSIXct(time,format="%d-%b-%Y %H:%M:%S",tz="EST"),
    # Break time into 1-minute intervals
    timecut = cut(time, breaks = "1 min"),
    timecutn = as.numeric(timecut)
  ) %>% 
   # Convert character types into numeric
  mutate_if(is.character, as.numeric) %>% 
  # Optional: filter data to time range of interest (e.g. time range where you've noticed something while driving)
  filter(time <= as.POSIXct("2018-10-13 19:25:00",tz="EST"))
```

Plots
-----

Once the 'trip2' data frame is prepared, you can start plotting your data. I've selected 11 variables of interest to plot: 
1. Engine RPM  
2. Oxygen Sensor @ Bank 1 Sensor 1 (Volts)  
3. Oxygen Sensor @ Bank 1 Sensor 2 (Volts)  
4. Air Fuel Ratio  
5. Short Term Fuel Trim @ Bank 1 (%)  
6. Fuel Trim @ Bank 1 Sensor 1 (%)  
7. Commanded Equivalence Ratio or Lambda  
8. Fuel Flow (litres/hour)  
9. EGR Error (%Error/%Commanded) 
10. Volumetric Efficiency  
11. Gas Mileage (litre per 100 km)

For more on what these mean, visit: [here for definitions](http://www.nology.com/supportedsensors.htm). I use ggplot2::ggplot to make nice and simple plots, all of which have a common x-axis.

``` r
# Function that will be called to draw polots
drawPlot <- function(df,x,y) {
  xx <- enquo(x)
  yy <- enquo(y)
  p <- df %>% 
    ggplot(aes_(x=xx, y=yy)) + 
    geom_line()
  return(p)
}

# List of column names that will be plotted
yvarlist <- lapply(c("rpm","o2b1s1_v","o2b1s2_v", "afr", "ftrimb1st_pc", "ftrimb1s1_pc", "lambda", "fflow_lph",
                     "egrcom_pc", "voleff_pc", "lp100k"),
                   as.name)

# For each column name in yvarlist, use the function drawPlot to create a plot
# Store the plots in list called 'p'
p <- lapply(yvarlist, drawPlot, df = trip2, x = time) 
q <- lapply(p, function(plotinlist){
  animated <- plotinlist + transition_reveal(as.POSIXct(trip2$time))
})
```

``` r
# Animated plot
lapply(q[1:2], plot)
```

![](/images/example_plot1.gif) ![](/images/example_plot2.gif)

``` r
# Static plots
p[[3]]
p[[4]]
```

![](/images/example_plot3.png) ![](/images/example_plot4.png)

Save plots as PDF
-----------------

You can save your plots as a .pdf file.

``` r
ggpubr::ggexport(plotlist = p, filename = "torqueR_Output.pdf", nrow = 4, ncol = 1)
print(paste0("PDF saved to: ",getwd()))
```

    ## [1] "PDF saved to: C:/Users/User/Documents/torqueR"
See what the final output looks like [here](/pdfs/torqueR_Output.pdf).