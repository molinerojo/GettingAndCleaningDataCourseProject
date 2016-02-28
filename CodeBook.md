#####Getting and Cleaning Data Course Project
Author: molinerojo

Date: 2016-02-28

**Description**

The purpose of this script is to collect, work with, and clean a data set for  
the task Course Project on the Coursera 'Getting and Cleaning Data' course.
The goal is to prepare tidy data that can be used for later analysis.
Next task will be performed:

1. Merges the training and the test sets to create one data set
2. Extracts only the measurements on the mean and standard deviation for each measurement.
3. Uses descriptive activity names to name the activities in the data set
4. Appropriately labels the data set with descriptive variable names. 
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

**Original Data Set**             
URL: https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip             
Description: 
The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. 
Each person performed six activities (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, 
LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and 
gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. 
The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly 
partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 
30% the test data. 
See 'README.txt' inside the zip file for more details.

**This script was created on the next environment**

R version:   3.2.3 (2015-12-10) -- "Wooden Christmas-Tree"

Platform:    x86_64-w64-mingw32/x64 (64-bit)

RStudio ver.:0.99.491 - © 2009-2015 RStudio, Inc.


**R-packages needed**

data.table

dplyr

reshape2


**DataSet Loading**

DownLoading the Data Sets file

    path       <- getwd()
    file       <- "Dataset.zip"
    path_file  <- paste(path,file,sep="/")
    
    fileURL    <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    download.file(fileURL, destfile=path_file)

Unzip the Data Sets file

    executable <- file.path("C:", "Program Files", "7-Zip", "7z.exe")
    parameters <- "x"
    cmd        <- paste(paste0("\"", executable, "\""), parameters, paste0("\"", path_file, "\""))
    system(cmd)

The files are unzipped in a folder call 'UCI HAR Dataset'

    path_ds <- file.path(path, "UCI HAR Dataset")

Subjects files Loading

    dtTrainSubjects <- read.table(file.path(path_ds, "train", "subject_train.txt"))
    dtTestSubjects  <- read.table(file.path(path_ds, "test",  "subject_test.txt"))

Activity files Loading

    dtTrainActivity_Y <- read.table(file.path(path_ds, "train", "Y_train.txt"))
    dtTestActivity_Y  <- read.table(file.path(path_ds, "test",  "Y_test.txt"))
    
    dtTrainActivity_X <- read.table(file.path(path_ds, "train", "X_train.txt"))
    dtTestActivity_X  <- read.table(file.path(path_ds, "test",  "X_test.txt"))

**TASK 1.Merges the training and the test sets to create one data set**

Merge Subjects

    dtSubjects   <- rbind(dtTrainSubjects, dtTestSubjects)
    setnames(dtSubjects, "V1", "subject")

Merge Activity Y

    dtActivity_Y <- rbind(dtTrainActivity_Y, dtTestActivity_Y)
    setnames(dtActivity_Y, "V1", "activityNum")

Merge Activity X

    dtActivity_X <- rbind(dtTrainActivity_X, dtTestActivity_X)

Create a new dataset with all the information

    dtTemp       <- cbind(dtSubjects, dtActivity_Y)
    dtAll        <- cbind(dtTemp, dtActivity_X)
    dtAll        <- data.table(dtAll)
    rm(dtTemp)

Order the new dataset

    setkey(dtAll, subject, activityNum)

**TASK 2.Extracts only the measurements on the mean and standard deviation for each measurement.**

Load the features File

    dtFeatures   <- read.table(file.path(path_ds, "features.txt"))
    setnames(dtFeatures, names(dtFeatures), c("featureNum", "featureName"))

Add a column for the FeatureCode 

    dtFeatures   <- mutate(dtFeatures, featureCode = paste0("V", featureNum, ""))

Filter the Features for Mean and SD

    dtFeaturesFiltered = subset(dtFeatures,grepl("mean\\(\\)|std\\(\\)",featureName))
    
    Temp <- c(key(dtAll), dtFeaturesFiltered$featureCode)
    dtAllFiltered <- dtAll[, Temp, with=FALSE]

**TASK 3.Uses descriptive activity names to name the activities in the data set**

Load the activity_labels File

    dtActivityNames <- read.table(file.path(path_ds, "activity_labels.txt"))
    setnames(dtActivityNames, names(dtActivityNames), c("activityNum", "activityName"))

Add Activity Names

    dtAllFiltered <- merge(dtAllFiltered, dtActivityNames, by="activityNum", all.x=TRUE)

**TASK 4.Appropriately labels the data set with descriptive variable names.**

I create a vector with the activitynum, the subject and the name of the filtered variables

    newNames <- c("activityNum","subject",as.vector(dtFeaturesFiltered$featureName),"activityName")

Remove the parenthesis characters on the labels

    newNames <- gsub("\\(\\)", "", newNames)

Replace the label of the variables on the datase with the created vector before.

    setnames(dtAllFiltered, names(dtAllFiltered), newNames)

**TASK 5.From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.**

Order the dataset

    setkey(dtAllFiltered, subject, activityName)

Delete the ActivityNum column no more needed

    dtAllFiltered$activityNum <- NULL

Define the field to Groupping and calculate the mean

    groupping               <- melt(dtAllFiltered,id.vars=c("subject","activityName"))
    dtAllFilteredSummarized <- dcast(groupping,subject + activityName ~ variable,mean)

Save the dataset to a text file tab delimited without rownames 

    write.table(dtAllFilteredSummarized, file.path(path,"TidyDataSet.txt"), sep="\t", row.names = F)

**Tidi Data Set Description**

Variable     | Type          | Values     | Description
------------ | ------------- | ------------- | -------------
subject | integer | from 1 to 30 | identifier of the subject corresponding to the measures
activityName  | character | LAYING, SITTING, STANDING, WALKING, WALKING_DOWNSTAIRS, WALKING_UPSTAIRS | Activity performed when the measure was taken
tBodyAcc-mean-X | numeric |  - | Average of the measures for the variable 'tBodyAcc-mean-X' for the subject during the activiy
tBodyAcc-mean-Y | numeric |  - | Average of the measures for the variable 'tBodyAcc-mean-Y' for the subject during the activiy
tBodyAcc-mean-Z | numeric |  - | Average of the measures for the variable 'tBodyAcc-mean-Z' for the subject during the activiy
tBodyAcc-std-X | numeric |  - | Average of the measures for the variable 'tBodyAcc-std-X' for the subject during the activiy
tBodyAcc-std-Y | numeric |  - | Average of the measures for the variable 'tBodyAcc-std-Y' for the subject during the activiy
tBodyAcc-std-Z | numeric |  - | Average of the measures for the variable 'tBodyAcc-std-Z' for the subject during the activiy
tGravityAcc-mean-X | numeric |  - | Average of the measures for the variable 'tGravityAcc-mean-X' for the subject during the activiy
tGravityAcc-mean-Y | numeric |  - | Average of the measures for the variable 'tGravityAcc-mean-Y' for the subject during the activiy
tGravityAcc-mean-Z | numeric |  - | Average of the measures for the variable 'tGravityAcc-mean-Z' for the subject during the activiy
tGravityAcc-std-X | numeric |  - | Average of the measures for the variable 'tGravityAcc-std-X' for the subject during the activiy
tGravityAcc-std-Y | numeric |  - | Average of the measures for the variable 'tGravityAcc-std-Y' for the subject during the activiy
tGravityAcc-std-Z | numeric |  - | Average of the measures for the variable 'tGravityAcc-std-Z' for the subject during the activiy
tBodyAccJerk-mean-X | numeric |  - | Average of the measures for the variable 'tBodyAccJerk-mean-X' for the subject during the activiy
tBodyAccJerk-mean-Y | numeric |  - | Average of the measures for the variable 'tBodyAccJerk-mean-Y' for the subject during the activiy
tBodyAccJerk-mean-Z | numeric |  - | Average of the measures for the variable 'tBodyAccJerk-mean-Z' for the subject during the activiy
tBodyAccJerk-std-X | numeric |  - | Average of the measures for the variable 'tBodyAccJerk-std-X' for the subject during the activiy
tBodyAccJerk-std-Y | numeric |  - | Average of the measures for the variable 'tBodyAccJerk-std-Y' for the subject during the activiy
tBodyAccJerk-std-Z | numeric |  - | Average of the measures for the variable 'tBodyAccJerk-std-Z' for the subject during the activiy
tBodyGyro-mean-X | numeric |  - | Average of the measures for the variable 'tBodyGyro-mean-X' for the subject during the activiy
tBodyGyro-mean-Y | numeric |  - | Average of the measures for the variable 'tBodyGyro-mean-Y' for the subject during the activiy
tBodyGyro-mean-Z | numeric |  - | Average of the measures for the variable 'tBodyGyro-mean-Z' for the subject during the activiy
tBodyGyro-std-X | numeric |  - | Average of the measures for the variable 'tBodyGyro-std-X' for the subject during the activiy
tBodyGyro-std-Y | numeric |  - | Average of the measures for the variable 'tBodyGyro-std-Y' for the subject during the activiy
tBodyGyro-std-Z | numeric |  - | Average of the measures for the variable 'tBodyGyro-std-Z' for the subject during the activiy
tBodyGyroJerk-mean-X | numeric |  - | Average of the measures for the variable 'tBodyGyroJerk-mean-X' for the subject during the activiy
tBodyGyroJerk-mean-Y | numeric |  - | Average of the measures for the variable 'tBodyGyroJerk-mean-Y' for the subject during the activiy
tBodyGyroJerk-mean-Z | numeric |  - | Average of the measures for the variable 'tBodyGyroJerk-mean-Z' for the subject during the activiy
tBodyGyroJerk-std-X | numeric |  - | Average of the measures for the variable 'tBodyGyroJerk-std-X' for the subject during the activiy
tBodyGyroJerk-std-Y | numeric |  - | Average of the measures for the variable 'tBodyGyroJerk-std-Y' for the subject during the activiy
tBodyGyroJerk-std-Z | numeric |  - | Average of the measures for the variable 'tBodyGyroJerk-std-Z' for the subject during the activiy
tBodyAccMag-mean | numeric |  - | Average of the measures for the variable 'tBodyAccMag-mean' for the subject during the activiy
tBodyAccMag-std | numeric |  - | Average of the measures for the variable 'tBodyAccMag-std' for the subject during the activiy
tGravityAccMag-mean | numeric |  - | Average of the measures for the variable 'tGravityAccMag-mean' for the subject during the activiy
tGravityAccMag-std | numeric |  - | Average of the measures for the variable 'tGravityAccMag-std' for the subject during the activiy
tBodyAccJerkMag-mean | numeric |  - | Average of the measures for the variable 'tBodyAccJerkMag-mean' for the subject during the activiy
tBodyAccJerkMag-std | numeric |  - | Average of the measures for the variable 'tBodyAccJerkMag-std' for the subject during the activiy
tBodyGyroMag-mean | numeric |  - | Average of the measures for the variable 'tBodyGyroMag-mean' for the subject during the activiy
tBodyGyroMag-std | numeric |  - | Average of the measures for the variable 'tBodyGyroMag-std' for the subject during the activiy
tBodyGyroJerkMag-mean | numeric |  - | Average of the measures for the variable 'tBodyGyroJerkMag-mean' for the subject during the activiy
tBodyGyroJerkMag-std | numeric |  - | Average of the measures for the variable 'tBodyGyroJerkMag-std' for the subject during the activiy
fBodyAcc-mean-X | numeric |  - | Average of the measures for the variable 'fBodyAcc-mean-X' for the subject during the activiy
fBodyAcc-mean-Y | numeric |  - | Average of the measures for the variable 'fBodyAcc-mean-Y' for the subject during the activiy
fBodyAcc-mean-Z | numeric |  - | Average of the measures for the variable 'fBodyAcc-mean-Z' for the subject during the activiy
fBodyAcc-std-X | numeric |  - | Average of the measures for the variable 'fBodyAcc-std-X' for the subject during the activiy
fBodyAcc-std-Y | numeric |  - | Average of the measures for the variable 'fBodyAcc-std-Y' for the subject during the activiy
fBodyAcc-std-Z | numeric |  - | Average of the measures for the variable 'fBodyAcc-std-Z' for the subject during the activiy
fBodyAccJerk-mean-X | numeric |  - | Average of the measures for the variable 'fBodyAccJerk-mean-X' for the subject during the activiy
fBodyAccJerk-mean-Y | numeric |  - | Average of the measures for the variable 'fBodyAccJerk-mean-Y' for the subject during the activiy
fBodyAccJerk-mean-Z | numeric |  - | Average of the measures for the variable 'fBodyAccJerk-mean-Z' for the subject during the activiy
fBodyAccJerk-std-X | numeric |  - | Average of the measures for the variable 'fBodyAccJerk-std-X' for the subject during the activiy
fBodyAccJerk-std-Y | numeric |  - | Average of the measures for the variable 'fBodyAccJerk-std-Y' for the subject during the activiy
fBodyAccJerk-std-Z | numeric |  - | Average of the measures for the variable 'fBodyAccJerk-std-Z' for the subject during the activiy
fBodyGyro-mean-X | numeric |  - | Average of the measures for the variable 'fBodyGyro-mean-X' for the subject during the activiy
fBodyGyro-mean-Y | numeric |  - | Average of the measures for the variable 'fBodyGyro-mean-Y' for the subject during the activiy
fBodyGyro-mean-Z | numeric |  - | Average of the measures for the variable 'fBodyGyro-mean-Z' for the subject during the activiy
fBodyGyro-std-X | numeric |  - | Average of the measures for the variable 'fBodyGyro-std-X' for the subject during the activiy
fBodyGyro-std-Y | numeric |  - | Average of the measures for the variable 'fBodyGyro-std-Y' for the subject during the activiy
fBodyGyro-std-Z | numeric |  - | Average of the measures for the variable 'fBodyGyro-std-Z' for the subject during the activiy
fBodyAccMag-mean | numeric |  - | Average of the measures for the variable 'fBodyAccMag-mean' for the subject during the activiy
fBodyAccMag-std | numeric |  - | Average of the measures for the variable 'fBodyAccMag-std' for the subject during the activiy
fBodyBodyAccJerkMag-mean | numeric |  - | Average of the measures for the variable 'fBodyBodyAccJerkMag-mean' for the subject during the activiy
fBodyBodyAccJerkMag-std | numeric |  - | Average of the measures for the variable 'fBodyBodyAccJerkMag-std' for the subject during the activiy
fBodyBodyGyroMag-mean | numeric |  - | Average of the measures for the variable 'fBodyBodyGyroMag-mean' for the subject during the activiy
fBodyBodyGyroMag-std | numeric |  - | Average of the measures for the variable 'fBodyBodyGyroMag-std' for the subject during the activiy
fBodyBodyGyroJerkMag-mean | numeric |  - | Average of the measures for the variable 'fBodyBodyGyroJerkMag-mean' for the subject during the activiy
fBodyBodyGyroJerkMag-std | numeric |  - | Average of the measures for the variable 'fBodyBodyGyroJerkMag-std' for the subject during the activiy


**For more detail about the measures, please see the file features_info.txt inside the zip file Dataset.zip**
