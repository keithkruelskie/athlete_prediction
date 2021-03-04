### Readme.md
### Capstone

### Problem Statement:
Modern race predictions for endurance athletes are often built on simple pace 
calculations, extrapolating race paces from a single point in time without 
incorporating training load and environmental factors. I aim to use machine 
learning on real world amateur athlete data to improve race prediction accuracy.

References:

https://exerciseinstitute.com.au/pace-calculator/

https://www.runnersworld.com/uk/training/a761676/rws-training-pace-calculator/

https://www.runnersworld.com/uk/training/a761681/rws-race-time-predictor/

### Executive Summary:

In order to train for running and other endurance events, many athletes choose  
to use a digital means of recording their training data. With the proliferation  
of digitized training devices (typically in the form of watches), many athletes  
now find it easy to visualize and analyze their data that they record themselves  
on runs, bikes, swims, or other activities.  
Coaches of these athletes, in order to monitor and suggest training improvements,  
can access this data and use it to improve their training plans, customizing  
training to the athlete rather than prescribing a "one size fits all" regimen.  
The coupling of the recording device and additional sensors, such as cadence,  
heart rate, and elevation sensors have let both coaches and athletes dial in  
their training to make them successful on race day.   
This project proposes that the data created by athletes in their daily and    
weekly training can be used to predict their performance on race day, a capability  
which can be highly valuable to coaches and athletes alike. Athletes, for instance,  
will be able to use this algorithm and a baseline training sample or samples,  
to accurately predict if, for example, they are on pace to qualify for Boston.  
Coaches can make use of this algorithm to compare athlete response to training  
stimuli, benchmark the athlete against previous season or previous race performances,  
and assess the performance of athletes they coach in a more general manner,  
gaining insight into their team and making sure the training is on track.

### Gathering Data:

Data was gathered from athletes willing to share their training files, and  
consisted of csv files extracted from training programs. Due to the added step  
and time required, it was decided to not attempt to use training program APIs  
to pull in data. Data was initially pulled by multiple athletes each from:

* Garmin (Garmin Connect)
* Strava
* Apple Health
* Training Peaks

Due to the time constraints on this initial phase of the project, and after  
reviewing and attempting to clean and slice the data, it was decided to  
focus on Garmin data as it had the most athletes and most complete sets of data  
(aside from TrainingPeaks). This would hopefully simplify cleaning and then  
additional features could be rolled out to other datasets with a generalized  
model. In total, the dataset contained 10,068 activities and 67 races. All of  
the data was organized by date, and so a time series analysis was a logical  
step to use machine learning. It was also decided that to focus on one sport  
initially (running) would yield the best usable model. Running also relies less  
on gear and aerodynamics than cycling, or specific skills such as in swimming,  
so it was chosen as the target sport.

### Cleaning and Visualizing:

Despite the relatively high amount of data that I was used to while training  
with my Garmin devices, the data provided by different devices is not consistent.  
Features in the datasets ranged from 29 to 47 columns, which meant that the cleaning    
portion would have to be specific about which variables were used later on for  
modeling. Some columns are only for cycling, which can be ignored as well.  
With that in mind, I started to visualize different variables to see if there  
was a more simple way to model, and one that might also be applicable to other  
sources of data such as Strava or TrainingPeaks.  
While visualizing the data it became apparent that there were a few things going  
on:

* Pace and heart rate are correlated
* Pace and distance are not necessarily correlated
* Distance and heart rate are not necessarily correlated

An engineered variable, intensity, was created to scale the runs. This brings the  
"pace" of recovery runs, and bad runs, up to an expected threshold pace, and  
also gives the model the flexibility to consider any distance run. Other variables  
to consider for future iterations may include time dependent variables, such as  
rolling average volume or total distance of runs, days until race, number of runs  
completed at a certain distance relative to race pace, and others.  
GAP is another important variable to consider when analyzing runs. GAP stands for    
"Grade Adjusted Pace" and is a metric that takes into consideration the elevation  
gain covered over the run. By using some trigonometry the corresponding flat ground  
pace can be calculated. This will make a large difference in training files which  
have a high amount of elevation gain and/or loss.

The data was parsed from raw files, which are continuous in terms of date, into  
individual files for each race to use for predicting. Files were created for 1  
month out, and 3 months out, to allow for comparison of the two time frames.

### Modeling:

Initially, a linear model was fit on athlete pace and distance. Due to the linear  
nature of many of the athlete's training and racing logs and the relative lack  
of change in pace and fitness in the final months before a race, it was relatively  
simple to find a reasonably accurate time (within a few seconds, to within a minute  
for most racers). For athletes that had a range of training intensities, or a  
seasonal component to their training, a straight linear approximation was not  
suitable, as the correlation between speed and distance was found to be positive  
in some cases. 

So, an autoARIMA method was implemented, to see if time and seasonal components  
were influencing final speed. By passing exogenous variables to the pmdarima   
autoARIMA model, accurate predictions were able to be obtained (within 10-15s/mile  
on most athletes). However, either due to the noise in the data or the relative  
strength of the exogenous variables, the ARIMA model most commonly selected a  
(0, 0, 0) parameterized model, which essentially means that a linear correlation  
existed for the pace and exogenous variables. Technically succes was achieved  
here, as we can accurately predict race times for a majority of our athletes  
with a few outliers, to a high degree of precision. 

Following the AR model, I decided to investigate the use of PyMC3 as a way of  
decomposing the "signal" in someone's threshold race pacing, and of using PyMC3  
to model the forecast created by an actual (1, 0, 1) time dependent SARIMAX model.  
I followed an example found in the statsmodels documentation: 

https://www.statsmodels.org/devel/examples/notebooks/generated/statespace_sarimax_pymc3.html  

And was able to predict one step ahead using the NUTS sampler. 

Not content with predicting just one day ahead, I implemented a PyMC3 decomposition  
of the seasonal trend found within several coached athlete's training files.  
By modeling the curve of pace from current day to race day as a sigmoid function,  
I was able to accurately predict 3 out of the 4 races from training files supplied  
by athletes who exhibited this seasonality. The seasonal decomposition also works  
on athletes without seasonality, as it begins to approximate a linear trend  
rather than a sigmoid shaped curve.With an athlete characteristic variable in the logistic  
equation, it should scale for each athlete's own abilities and response to training.

So given an athlete's previous training file for a race, current average pace/  
intensity, and a target race date and distance, my model was able to accurately  
predict within :10-15s from a time frame of 5-20 weeks out from race day. More  
testing is needed, as there are still some external factors to consider (such as  
running surface) but the model appears to generalize well for athletes that consistently  
train in a consistent manner. 


### Conclusions:

In conclusion, I had a great time diving into the nuts and bolts of what makes  
racing so hard to predict. Not every athlete has a consistent seasonal component.   
Not every running surface is created equal. I plan on developing this model further  
and turning it into an app of sorts, possibly with Strava or Garmin integration.

Main takeaways are as follows:
1. Real world data is messy, and significant energy is spent to get it into a format  
that can be used
2. You get out what you put in - when it comes to training, consistency breeds  
more consistency. 
3. Coached athletes tended to have a more pronounced seasonality to their training  
data (though the sample size should be increased)
4. Predictions for race times held true when applying the athlete's training trend  
to races later in the season


