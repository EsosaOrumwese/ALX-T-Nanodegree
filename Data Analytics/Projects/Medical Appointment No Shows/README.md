# Exploratory Data Analysis on Medical Appointment No Shows
## Introduction
Why do patients miss their scheduled medical appointments? This project looks at the variables surrounding patients who have made appointments and attempts at gaining insights as to why some missed their appointment in a hospital in Brazil.


| Attributes | Description|
| ----- | ----- |
| `PatientID` | Identification of a patient |
| `AppointmentID` | Identification of each appointment |
| `Gender` | Male or Female |
| `ScheduledDay` | The day someone called or registered the appointment (This is before the appointment) |
| `AppointmentDay` | The day of appointment  when they have to visit the doctor.|
| `Age` | Age of the patient |
| `Neighborhood` | Where the appointment takes place |
| `Scholarship` | If the patient is a partaker of the [Bolsa Família](https://en.wikipedia.org/wiki/Bolsa_Fam%C3%ADlia), a social welfare program of the Government of Brazil. |
| `Hypertension` | Whether the patient has hypertension. |
| `Diabetes` | Whether the patient has diabetes |
| `Alcoholism` | Whether the patient is an alcoholic |
| `Handicap` | Whether the patient is handicapped |
| `SMS_received` | Whether 1 or more messages were sent to the patient |
| `No_show` | Did the patient show up? |

`No_show` is our variable of interest. 


## Inferences and Conclusion
From my analysis, I was able to discover that a large number of patients in this dataset didn't have any of the listed ailments and were non alcoholics. Most of the patients most likely paid their medical bills themselves since a large number of them weren't on the Bolsa Família scholarship program. I also noticed that most of the patients in this data were women as women generally take good care of their health than men do. Also while there's a similarity between the number of those who received an SMS and those who didn't, we can see that a majority of the people didn't show up regardless of whether they got an SMS or not.

Let's revisit the questions that were posed earlier on.

**Q1. What factors are important for us to know in order to predict if a patient will show up for their scheduled appointment?**

From the heatmap of the correlations between all the variables and the target variable (`No_show_numeric`), I noticed that most of our predictor variables had a weak (~0) relationship with the target variable. So far `Scholarship`, `Alcoholism` and `timeToAppointment` had the relatively highest correlation with the target variable, with 0.045, 0.02 and 0.06 respectively.

**Q2. What day of the week to do most people miss their appointment? Consequently, what day do most people show up?**

From my analysis, I found out that Tuesday had the highest percentage (23.0%) of missed appointments followed by Wednesday (22.5%) then Monday (21.5%). This is probably due to the fact that these are the busiest days of the week. Hence people tend to be preoccupied leading to a no-show. Saturday recorded an approximately 0.0% no_show. Only 9 people from this dataset missed their appointments. 

When looking at the days that patients showed the most, I found out that Wednesday had the highest percentage (24.1%) of patients showing up for their appointments. Next was Tuesday with 22.8% then Monday with 19.8%. I noticed that Saturday also had the lowest percentage with only 22 patients showing up in this dataset. 

I was able to infer that the first 3 days of the week are the busiest and also the days that are most likely to get the highest no shows.

**Q3. What is the general age of persons who tend to miss their appointments?**

After exploring this data, I noticed that majority of patients who are no-shows were between the age of 0 to 60 with the mean age being 34.8years while for those who attended their appointments had a peak age of around 53yrs with the mean age being 39.95 yrs. There are no outliers in this variable. 

**Q4. Do SMS reminders affect the average attendance?**

From the data, I didn't see the effect that SMS reminders had on the average attendance. It neither boosted it nor retarded it. Analysis of those who didn't receive an SMS showed that a majority (70.6%) of the patients did not show up and only 29.4% made it for their appointments. From those who received an SMS reminder, 72.4% still didn't show up.

**Q5. What neighborhood is famous for missing their appointments?**

Jardim Camburi was seen to have the highest percentage of no shows (6.6%) when compared to the rest of the neighborhoods of no shows to appointments. They were followed by Maria Ortiz which has 5.6% of the total no shows.


## Limitations
* Since most of the variables were binary it nature, I wasn't able to draw insights on how they related to target variable in question (`No_show`).
* Other variables like *'distance between patient's home and hospital'*, *'is the patient experiencing any actute symptoms'*, *`patient's perceived wait time in the hospital'*, *'patient's perceived service quality'*, etc, can be taken into consideration during data collection.


## References
* The dataset was gotten from [Kaggle](https://www.kaggle.com/datasets/joniarroba/noshowappointments).