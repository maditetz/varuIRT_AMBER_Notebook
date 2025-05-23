# AMBER Tutorial

## Before Getting Started
You can use the provided example videos or your own videos to complete this tutorial. The example video files can be downloaded from our OSF repository ([OSF example videos](https://osf.io/e3dyc/)) from the "example_videos" folder.

If you are using your own videos, make sure all the videos you want to analyze at once are in their own directory on your computer. The pose estimation will be run on all the video files in the folder you specify. Move any videos you do not want to run to another location.

If you are wondering about the best way to record videos to be compatible with AMBER, check out the [Video Recording information page](https://github.com/lapphe/AMBER-pipeline/wiki/Video-Recording).

Make sure you have followed the instructions for installing and setting up:
- DeepLabCut
- SimBA
- AMBER files from this GitHub repository
- Behavior classifiers from the OSF repository

Detailed installation and set up instructions can be found [here](https://github.com/lapphe/AMBER-pipeline/wiki/Installations-and-set-up).

# Pose Estimation

The AMBER_pose_estimation.py script will run your videos through all pose estimation and post-pose estimation steps required for all videos in the video folder. It will then prepare files for use in SimBA.

The script will automatically run the following steps:
1. Pose estimation for dams for all videos using DeepLabCut and the AMBER dam pose estimation model
2. Create videos to check dam tracking
3. Pose estimation for pups for all videos using DeepLabCut and the AMBER pup multi-animal pose estimation model
4. Create videos to check pup detections
5. "Unpickle" pup detection files to convert to CSV
6. Join and reformat pup and dam pose estimation output so it is ready to use with SimBA

## Run Pose Estimation Steps

1. Open the Windows command prompt with administrator privileges

2. Activate your deeplabcut conda environment <br> 
``conda activate DEEPLABCUT``

3. Move to the AMBER-pipeline directory <br>
Change your current directory so you are in the AMBER-pipeline directory containing all the files downloaded when you cloned the AMBER repository using `cd /d path/to/directory` on Windows
<br> e.g. if the AMBER-pipeline folder is located on the desktop: `cd /d C:\Desktop\AMBER-pipeline`

4. Make sure all the videos you want to run are located in a single folder <br>
They can be anywhere on your computer -they do not need to be in the AMBER-pipeline folder. Copy the address of the folder containing the videos to run. <br>
   _Note: in Windows, you can copy the directory path by right-clicking on the folder name in the file explorer and selection “Copy address as text”. You can then paste it in the the command window_

5. Run pose estimation steps <br>
To run pose estimation, you will enter “python”, the script name,  followed by the path to the directory where your videos are location. <br>
e.g. `python AMBER_pose_estimation.py C:\Desktop\example_videos` <br>
Press enter to execute the command.<br>
The deeplabcut files will appear in the same directory as your videos. There will also be **two new folders created**: <br>
The first folder, _pose_estimation_videos_, contains the video with the labeled dam and pup points to check model performance. These videos have been move to a separate directory to make importing videos into SimBA easier later on. <br>
The second folder, _AMBER_joined_pose_estimation_, contains the reformatted and combined pose estimation files with dam and pup tracking that are used during behavior classification in SimBA. 
 
6. Check your pose estimation videos 
Check the labeled videos to ensure that the tracking looks good before proceeding to behavior classification. If the pose estimation models are not performing well, you may need to label additional frames from your videos and retrain the dam or pup models. Note that pose estimation does not need to be perfect to get accurate behavior classification, but major tracking mistakes will reduce the performance of the classifiers. 
If you feel confident in the pose estimation model performance on your videos and want to skip the “create tracking videos” step, you can add the “skip_create_videos” argument when you run the script.  <br>
e.g. `Python AMBER_pose_estimation.py C:\Desktop\hannah_test_short skip_create_videos`

7. Exit your deeplabcut conda environment** <br>
`conda deactivate`

# Behavior classification <br>
Behavior classification is performed in SimBA using the preconfigure AMBER_SimBA_project. <br>

1. Start the SimBA conda environment and open simba<br>
`conda activate simbaenv` <br>
`simba` 
2. Load the SimBA_AMBER_project using the SimBA GUI. <br>
The project config file is found in _AMBER-pipeline/SimBA_AMBER_project/project_folder/project_config.ini_. From here, you can follow the SimBA user [guide for analyzing new videos](https://github.com/sgoldenlab/simba/blob/master/docs/Scenario2.md) or follow the steps below. <br>

3. Import videos <br> 
These should be the same videos you performd pose estimation on. You can select the "Import SYMLINK" box to use symbolic links to the videos instead of copying the full videos into the SimBA project.

4. Import tracking data <br>
These csv files can be found in the “AMBER_joined_pose_estimation” folder created during pose estimation. 
5. Set video parameters <br>
As described in the [Video recording section](https://github.com/lapphe/AMBER-pipeline/wiki/Video-Recording "Video recording for AMBER"), we used the know distance of the food hopper, which is positioned about halfway of the depth of the cage (see images on under Video Recording). As a side-view recording, distances calculated will not be completely accurate since the actual distance varies depending on the location of the animal in the cage. However, setting these known distances helps account for variation in recording resolution and the distance from the cage to the front of the cage. Ifyour cage set up is different, you can select a different know distance visible in your videos, although we suggst selecting something that is about at the mid point of the cage depth. 

6. Skip outlier correction <br>
We recommend skipping outlier correction because this step relies on body-length distance across all frames to perform these calculations, which is influenced by the dramatic differences in body length when the dam is near the front versus back of the cage.<br>
_(Don’t forget to actually tell SimBA to skip this step on the Outlier Correction tab! This will ensure the csv files are copied over to the new location and can be used ifor feature extraction in the next step.)_ 
7. Extract features <br>
Select "Apply user-defined feature extraction script" and use the customized AMBER feature extraction script. This script is located in AMBER-pipline/SimBA_AMBER_project/AMBER_feature_extraction/amber_feature_extraction.py<br>
![extract features](https://user-images.githubusercontent.com/53009913/232091989-cd38972c-6d97-4248-b5c8-2384bc7938e5.png)

_Note: This step can take a long time for long videos. The convex hull and back circle fitting calculations take a lot of computational time, but are among the most important features for several behavioral classifiers. For an hour long video recorded at 30fps, this step takes about 25 minutes per video, however, run time will vary depending on your computer specs._ 
<br>
<br>
**(Skip the "Label behavior" and "Train machine models" steps. Those steps are used for creating new behavior classifier models. We will use previously created models)** 

8. Run the machine models <br>
It’s a good idea to [validate the provided models on your videos](https://github.com/sgoldenlab/simba/blob/master/docs/validation_tutorial.md) on your videos* and determine a good discrimination threshold for each classifier before running the models on all of your videos. Below are discrimination thresholds that work well for the example videos, but you should confirm performance with your own videos. <br>
![threshold table for github](https://github.com/lapphe/AMBER-pipeline/assets/53009913/9120f595-45ee-49f3-954c-f586506fad21)

9. Analyze all of your videos <br>
Find the models (they were moved to _AMBER-pipline/SimBA_AMBER_project/models_ during set up) and then enter the discrimination threshold and minimum bout length for analysis. Click “Run models”. <br>
### Congratulations, you now have maternal behavior annotations!

SimBA provides several tools for post-classification analysis and [visualizations](https://github.com/sgoldenlab/simba/blob/master/docs/visualizations_tutorial.md) that can be used with your data. Or, you can use the csv files found in _?AMBER_SimBA_project/project_folder/csv/machine_results_ that contain behavior lables for each frame along with the extracted features and pose estimation coordinates for all body parts. 
