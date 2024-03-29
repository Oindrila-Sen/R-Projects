Titanic - What would have been my probability of survival if I had boarded that ship?
-----------------------------------------------------------------------------------------

# Introduction

I was in my teens when I first came to know about Titanic.I sat in between my parents in the Movie Theatre, for the first time, to watch a Hollywood Blockbuster. I was mesmerized by the grandeur, the emotions, and the sadness. For the first time, I realized a ship named "Titanic" existed for real. It sailed some day on April 1912 and never returned. On my way back home from the Theatre, I imagined myself on that ship. The Titanic fever continued for a few more days. 

After 20 years of that unforgettable experience, I got a chance to re-live that experience. My first Kaggle entry is the Titanic Dataset where I would explore if I had boarded the ship, what would have been my chances of survival?

Let's Load the data and start digging!

# Load Data

```{r}
library(dplyr)
library(ggplot2)
######################################
# 1. Load Data
######################################
# Set Working Directory
setwd("/Users/oindrilasen/WORK_AREA/Data Science/kaggle/Titanic")
# Read train.csv data file
titanic_train<-read.csv("train.csv",
                  header = TRUE,
                  na.strings = "",
                  stringsAsFactors = FALSE)

# Read test.csv data file
titanic_test<-read.csv("test.csv",
                        header = TRUE,
                        na.strings = "",
                        stringsAsFactors = FALSE)

# Add column Survived in Test DataSet
titanic_test$Survived <- NA
titanic_clean <- rbind(titanic_train,titanic_test)
```
Now, Let's take a look at the data.
```{r}
dim(titanic_train) # 891,12
```
![Alt text](Titanic_files/figure-html/1.png)

```{r}
dim(titanic_test) # 418, 11
```
![Alt text](Titanic_files/figure-html/2.png)

```{r}
glimpse(titanic_clean) # 1309, 12
```
![Alt text](Titanic_files/figure-html/3.png)

# Data Wrangling & Cleaning

1. #### Convert to Factors:

First let's see which variables are more likely to be a Factor and then convert those variable to a Factor Data Type.
```{r}
# Check which variables are Factors
sapply(titanic_clean, function(x) length(unique(x)))
```
![Alt text](Titanic_files/figure-html/4.png)

We will transform the below variables to Factors:
1. Survived
2. Pclass
3. Sex
4. Embarked

```{r}
# Transforming categorical Variables to factors:
to_factor <- c(
  'Survived',
  'Pclass',
  'Sex',
  'Embarked'
)
for (col in to_factor) {
  titanic_clean[[col]] <- factor(titanic_clean[[col]])
}
```
2. #### Check for NA records:

Before proceeding any further, let's find out which variables have NA values.
```{r}
# check for NA values
sapply(titanic_clean, function(x) sum(is.na(x)))
```
![Alt text](Titanic_files/figure-html/5.png)

Now, we will replace the NA value for each variable with some meaningful values.

* Age: First, let's convert the Age variable to Integer and replace the NA values with the Mean of the Age data.
```{r}
# Convert Age column to Numeric
titanic_clean$Age <- as.integer((titanic_clean$Age))
# Relace NA values for Age with the mean
titanic_clean$Age[is.na(titanic_clean$Age)] <- mean(titanic_clean$Age,na.rm = TRUE)
```
* Cabin#:There are a couple of records where the Cabin# is not populated. Let's replace those records with "None".
```{r}
# Replace Cabin# with None for NA records
titanic_clean$Cabin[is.na(titanic_clean$Cabin)] <- "None"
```
* Embarked: There are three current values for Embarkation. Let's replace the NA values with the most common value as per our current Dataset.
```{r}
# Check for Embarked variable
table(titanic_clean$Embarked)
# Relace the Embarked value with the most common value i.e S
titanic_clean$Embarked[is.na(titanic_clean$Embarked)] <- "S"
```
* Fare: Fare has some NA values and we will replace those with the MEAN value of the Fare data.
```{r}
# Convert Fare column to Integer
titanic_clean$Fare <- as.integer((titanic_clean$Fare))
# Relace NA values for Fare with the Mean
titanic_clean$Fare[is.na(titanic_clean$Fare)] <- mean(titanic_clean$Fare,na.rm = TRUE)
```
Let's check again for NA records.
```{r}
# Again check for NA values
sapply(titanic_clean, function(x) sum(is.na(x)))
```
![Alt text](Titanic_files/figure-html/6.png)

3. #### Add meaning to the data:
```{r}
# Change the levels to meaningful values
# 1. Pclass
levels(titanic_clean$Pclass)[levels(titanic_clean$Pclass)== "1"] <- "1st Class"
levels(titanic_clean$Pclass)[levels(titanic_clean$Pclass)== "2"] <- "2nd Class"
levels(titanic_clean$Pclass)[levels(titanic_clean$Pclass)== "3"] <- "3rd Class"

# 2. Embarked
levels(titanic_clean$Embarked)[levels(titanic_clean$Embarked)== "C"] <- "Cherbourg"
levels(titanic_clean$Embarked)[levels(titanic_clean$Embarked)== "Q"] <- "Queenstown"
levels(titanic_clean$Embarked)[levels(titanic_clean$Embarked)== "S"] <- "Southampton"
```
# Add New Features

####  Fare Group: 
It looks like the Fare for the ship ranges from 0 to $512. Let's create a new field Fare_Group with values "Low", "Medium" and "High" as per the Fare value.
```{r}
summary(titanic_clean$Fare)
titanic_clean$Fare_Group <-factor(ifelse(titanic_clean$Fare >= 0 & titanic_clean$Fare <= 15, "Low",
                           ifelse(titanic_clean$Fare > 15 & titanic_clean$Fare <=100, "Medium",
                           ifelse(titanic_clean$Fare >100 ,"High",NA
                                               ))))

```
#### Age_Group: 
Again, Age varies from an infant less than a year to an old person of around 80 years of age. So, add a new Age_Group column with values like "Baby","Kid","Teen" and "Adult".
```{r}
# Add new feature Age_Group
summary(titanic_clean$Age)

titanic_clean$Age_Group <-factor(ifelse(titanic_clean$Age<= 3, "Baby",
                          ifelse(titanic_clean$Age> 3 & titanic_clean$Age<=12, "Kid",
                          ifelse(titanic_clean$Age> 12 & titanic_clean$Age<=18, "Teen",
                          ifelse(titanic_clean$Age> 18, "Adult",NA
                                 ))))
)
```
#### With_Family: 
As per the Dataset, if there is a value other than zero(0) in the fields "SibSp" and "Parch", the passenger is with a Family.
```{r}
# Add new feature with_family
titanic_clean$with_family <-factor(ifelse(titanic_clean$Parch == 0 & titanic_clean$SibSp ==0, "no","yes"))
```

# Exploratory Data Analysis

#### 1.Class vs Total Passengers
```{r}
# Understand the Variables Individually
ggplot(titanic_clean, aes(x = Pclass)) +
  geom_bar(fill= "light blue")+
  scale_y_continuous(limits = c(0,1400), breaks = seq(0,1400,100))
  ggtitle("Class vs Total Passengers") 
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-13-1.png)

It looks like most of the passengers were travelling in the 3rd Class. But,1st class passengers count was a little more than that of the 2nd Class.

#### 2.Sex vs Total Passengers
```{r}
prop.table((table(titanic_clean$Sex)))
```
![Alt text](Titanic_files/figure-html/13.png)

```{r}
ggplot(titanic_clean, aes(x = Sex)) +
    geom_bar(fill= "light blue")+
    scale_y_continuous(limits = c(0,1400), breaks = seq(0,1400,100))
  ggtitle("Sex vs Total Passengers") 
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-14-1.png)

Around 65% of the passengers were Male and only 35% were Female. 

#### 3.Embarked vs Total Passengers:
```{r}
ggplot(titanic_clean, aes(x = Embarked)) +
    geom_bar(fill= "light blue")+
    scale_y_continuous(limits = c(0,1400), breaks = seq(0,1400,100))
  ggtitle("Embarked vs Total Passengers") 
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-15-1.png)

Most of the Passengers boarded the ship at South Hampton. Around 250 passenegrs boarded at Cherbourg and around 120 boarded at Queenstown.

#### 4.Age_Group vs Total Passengers
```{r}
ggplot(titanic_clean, aes(x = Age_Group)) +
    geom_bar(fill= "light blue")+
    scale_y_continuous(limits = c(0,1400), breaks = seq(0,1400,100))
  ggtitle("Age_Group vs Total Passengers") 
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-16-1.png)

Most of the passenegrs were Adult which is obvious. But, there were a few babies and kids too.

#### 5.Fare_Group vs Total Passengers
```{r}
ggplot(titanic_clean, aes(x = Fare_Group)) +
    geom_bar(fill= "light blue")+
    scale_y_continuous(limits = c(0,1000), breaks = seq(0,1000,100))
  ggtitle("Fare_Group vs Total Passengers") 
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-17-1.png)

Most of the passengers were in the "Low" Fare group. It tallys with the Data where we have seen before that most of the passengers were in the 3rd Class.

#### 6.with_family vs Total Passengers
```{r}
prop.table(table(titanic_clean$with_family))
```
![Alt text](Titanic_files/figure-html/10.png)

```{r}
ggplot(titanic_clean, aes(x = with_family)) +
    geom_bar(fill= "light blue")+
    scale_y_continuous(limits = c(0,1000), breaks = seq(0,1000,100))
  ggtitle("with_family vs Total Passengers")   
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-18-1.png)

It looks like 60% of the passengers were travelling alone and 30% were with a Family.

#### 7.Kids tarvelling alone?!
```{r}
  table(titanic_clean$with_family, titanic_clean$Age_Group)
  ```
  ![Alt text](Titanic_files/figure-html/9.png)
  
  ```{r}
   kids_without_family<-
    titanic_clean%>%
    filter(with_family=="no",Age_Group=="Kid")
  
  kids_without_family
```
That's strange! 3 kids were travelling alone? Where were their parents or Guardians?

Now, let's explore the survival rates. For this, we would need the records with a value in Survived column. So, let's divide the current Dataset as before.
```{r}
set.seed(550)
titanic_train <- titanic_clean[1:891,]
titanic_test <- titanic_clean[892:1309,]
```
#### 8.Age_group vs Survived
```{r}
table(titanic_train$Survived,titanic_train$Age_Group)
ggplot(titanic_train, aes(Age_Group, ..count..)) + 
geom_bar(aes(fill = Survived), position = "dodge", na.rm = FALSE)
  ```
![Alt text](Titanic_files/figure-html/unnamed-chunk-21-1.png)

There were mostly Adults in that ship and many Adults did not make it. But what's sad is that there were 10 Babies and 19   Kids who did not survived.

#### 9.Sex Vs Survived

```{r}
table(titanic_train$Survived,titanic_train$Sex)
ggplot(titanic_train, aes(Sex, ..count..)) + 
  geom_bar(aes(fill = Survived), position = "dodge", na.rm = FALSE)
```
![Alt text](Titanic_files/figure-html/unnamed-chunk-22-1.png)

It looks like the Female passengers survived more than the Male passengers.

#### 10.With_Family vs Survived
```{r}
table(titanic_train$Survived,titanic_train$with_family)
ggplot(titanic_train, aes(with_family, ..count..)) + 
  geom_bar(aes(fill = Survived), position = "dodge", na.rm = FALSE)
  ```
  ![Alt text](Titanic_files/figure-html/unnamed-chunk-23-1.png)
  
  The passengers with Family were less. But, it looks like that the survival rate for the passengers with family was most likely. Does that sound logical? Ohh yes!
  
 #### 11.Fare_Group Vs Survived
 
  ```{r}
prop.table(table(titanic_train$Survived,titanic_train$Fare_Group))
ggplot(titanic_train, aes(Fare_Group, ..count..)) + 
  geom_bar(aes(fill = Survived), position = "dodge", na.rm = FALSE)
  ```
![Alt text](Titanic_files/figure-html/unnamed-chunk-24-1.png)

It looks like the passengers with high fare had more survival probability. The passengers with Medium Fare had a 50:50 chance. The Lowest Fare group were the victims.

#### After seeing all these results, I really want to find out, what was my chance of survival, if I had boarded that Ship?

Since, it is a hypothetical situation, Let's go back 20 years of my life when I first asked that question.So, 20 years back, I was a Teenager and I wold have travelled with my parents and my sister.

```{r}
my_chances <-
  titanic_train %>%
  filter(Sex=="female",
         Age_Group == "Teen",
         Parch >0 ,
         SibSp >0 
         )
prop.table(table(my_chances$Survived))
```
![Alt text](Titanic_files/figure-html/11.png)

#### Phew! It looks like I had a 75% chance to survive! 

# Create a Model

Since, I am new to Data Science or Machine Learning, I have a very limited Knowledge on which Model to select for prediction in the current scenario. As per my knowledge, to predict a categorical variable, we can use a Logistic Regression Model and here it is.

```{r}
######################################
# 5. Create a Model
######################################
lm_survival_model <- glm(Survived ~ Pclass+Sex+
                                    Age+SibSp+Parch
                           +with_family+Age_Group,
                           data = titanic_train,
                           family = binomial(link=logit)
)

summary(lm_survival_model)
```
![Alt text](Titanic_files/figure-html/12.png)

Now, let's use this model to predict the Survived column value for our Test DataSet.
```{r}
# Prediction
predict_survival <- round(predict(lm_survival_model,
                                  titanic_test,type =  "response"))
titanic_test$Survived <- predict_survival
table(titanic_test$Survived)
```
![Alt text](Titanic_files/figure-html/14.png)

#### Well, my Model says that 63% of the Test data passengers did not survived and only 37% did survived.That's a Sad result!

```{r}
# Write the Final Solution
final_solution <- titanic_test%>%
                  select(PassengerId,Survived)
write.csv(final_solution, 
          file = "final_solution.csv")
```
That's all I have for today!
Thank You for reading!
                                
