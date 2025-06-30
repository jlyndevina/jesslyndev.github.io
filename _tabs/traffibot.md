---
layout: page
title: Traffibot Project
permalink: /projects/traffibot/
order: 7
---

# TraffiBot: ROS & ML Autonomous Traffic Control Robot 🚦

*By: Jesslyn Devina and Eldad Zipori*

<div class="project-hero-image">
  <img src="/assets/img/projects/traffibot/tb_1.png" alt="LR 92 corners" class="project-image">
</div>

## Introduction

ENPH 353 is an introductory machine learning course for engineering physics students in UBC. For our final project, we are required to develop software for an autonomous robot in ROS where it can perform several tasks such as driving autonomously, reading license plates, identifying pedestrians and obeying traffic regulations.

In this paper, we will explain our methods. Our approach can be separated into 2 main modules:

- **Driving** - Autonomous navigation using computer vision
- **License plate recognition** - OCR using CNN for character detection

---

## Drive 🚗

For the driving mechanism of the car we have decided to apply a finite state machine that will relay on computer vision data obtained from the image feed at the front of the car. The initialization of the state machine was done very loosely for the sake of time and due to lake of proper advance preparation. The state finite state machine and the determination of the action to take was done with various Boolean flags (Figure 1) and if-else statement.

### State Machine Implementation

```python
"""
State Machine Flags
"""
PID_DRIVE_FLAG = False
RED_LINE_FLAG = False
ENTER_CENTER = False
```

As the robot performs operations of the current state, it gathers data to update the various flags and change the state of the state machine.

### State Machine Flow

As seen in Appendix A, the state machine starts by automatically driving the first stroke of competition surface. The reasoning behind this is the complication associated with deciding whether to take a right or a left in the beginning. A generic approach that can be taken in the case where the robot is capable to both determine and drive the SFU hill. In that case in a case where the robot sees no white line both from right and left under the white color mask, it will take a random turn, left or right, and will be able to drive through the competition surface independent of that decision.

The second state of the finite state machine is **PID driving** using a white mask on the image feed from the front camera. The PID algorithm calculated the distance from the right white line in on the competition surface. The PID algorithm tries to keep the robot 300px away from the line. In the case the right line is not detected the algorithm will try finding the left white line in order to keep the robot in the center of the road.

In addition to the white mask information to center the robot on the road, during the PID state the robot gets a constant feed of red line mask to find the stop line in front of a stop walk. Because of the use of a state machine using flags, the robot is able to be in two states at once. In this case PID and red line detection. During the red line detection state the algorithm determines the distance from a clear red line right in the center of the image feed to the robot. It is worth noting that the red lines are very clear under the mask and thus very affective. As the robot determines it is right in front of the line the state machine goes out of the red line detection state, to avoid stopping due to the second red line, and out of the PID detection state into the pedestrian crossing state.

In the **pedestrian crossing state** the algorithm detect the initial location of the pedestrian using openCV HOGDescriptor default people detector. As the algorithm notes the initial location of the pedestrian it keeps reading it location until the pedestrian is across the white line on the other side of the road, i.e. if the pedestrian was closer to the right the algorithm will wait until the pedestrian is a few pixels from the left of the left white line. After the detection that the pedestrian crosses the robot will drive one second straight and then turns on the PID detection and the red line detection.

Finally, after the second pedestrian crossing, the algorithm goes into the **SFU hill state** where it drives through it automatically using a hard code of the surface. It should be noted that this case exists as we did not have time to find the appropriate detection algorithm for SFU hill. We have started looking into driving through SFU hill using the right line of the sky as it is very clear to detect. Under these circumstances there will be no reason to count the amount of pedestrian detected so far, as the PID algorithm will go into a different state when it cannot find an appropriate white line both on right and left.

---

## License Plate Recognition 🔍

As the robot traverses around the environment, there are several cars parked along the way. There are several methods to identify the individual license plates of the car. Our final approach is a blend of image processing (masking, morphing), contours and perspective transform.

### Methodology

Here is our step-by-step approach:

1. **Recognizing that the cars are all the same colour (blue)** so we mask for the blues (2 rectangular corners) of the car.
   - Threshold changes due to sunlight so we have 2 different shades of blue to select from.
   - Transform/modify the camera image into a hsv image and mask it to extract only the blue rectangles.

2. **Finding the two largest contours**
   - The targeted car is not the only object blue in colour. Other cars in the background and the sky can be mistaken during contour detection. Thus, we implemented several restrictions on the contours:
     - Ensure that the 2 contours have similar height and their y axis values have some overlap.
     - Ensure that the area of at least one of the two contours are large enough (more than 2% of the entire image) before trying to read the license plate to improve accuracy.

3. **After obtaining the two largest contours**, we obtain the 4 coordinates of the grey rectangle between those contours.

4. **Apply perspective transform using homography.**

<div class="image-gallery" style="display: flex; flex-wrap: wrap; gap: 16px; justify-content: center;">    <div class="image-item" style="display: flex; flex-direction: column; justify-content: flex-end; align-items: center; height: 250px; text-align: center;">
    <img src="/assets/img/projects/traffibot/tb_2.png" alt="Finding the 2 largest blue contours" style="max-height: 180px; object-fit: contain;">
    <p style="margin-top: 8px;"><em>Fig 2: Finding the 2 largest blue contours</em></p>
  </div>

  <div class="image-item" style="display: flex; flex-direction: column; justify-content: flex-end; align-items: center; height: 250px; text-align: center;">
    <img src="/assets/img/projects/traffibot/tb_3.png" alt="Perspective transform" style="max-height: 180px; object-fit: contain;">
    <p style="margin-top: 8px;"><em>Fig 3: Perspective transform</em></p>
  </div>

  <div class="image-item" style="display: flex; flex-direction: column; justify-content: flex-end; align-items: center; height: 250px; text-align: center;">
    <img src="/assets/img/projects/traffibot/tb_4.png" alt="Masking the license plate" style="max-height: 180px; object-fit: contain;">
    <p style="margin-top: 8px;"><em>Fig 4: Masking the license plate</em></p>
  </div>
</div>

5. **Reading the license plate and parking position.**

   **License Plate**
   - Crop the bottom half of the image where the license plate is, mask the image for the blues again, and find the 4 largest contours that represents the 4 characters on the license plate.
   - Sometimes masking does not work well and can split the characters into multiple contours so we applied restrictions:
     - Ensure that the areas of the smallest contours found is more than 30% of the largest
   - Crop each character into the same size.
   - Run them through the CNN for character recognition

   **Parking Position**
   - Do the same procedure as reading the license plate but use the top half instead of the bottom half of the image and find the 2 largest contours instead of 4.
   - Since we know that the first character will always be 'P', we will only read the second character which is the parking position

### The CNN Model 🧠

To read the license plate, we use a convolutional neural network to identify the characters in the plate. The CNN architecture is identical to Lab 6.

#### Architecture

Our CNN model has 4 convolutional layers, followed by max-pooling layers and two dense layers.

**Structure Summary:**
- A 2D convolutional later layer with 32 filters of size 3x3 and a ReLU activation function.
- A Max Pooling 2D layer that down samples the output of the first layer by taking the maximum value in a 2x2 window.
- Then we apply 3 more layers of convolutional layer with 64 filters of size 3x3 and a ReLU activation function followed by a max-pooling layer in a 2x2 window.
- A Flatten later that squeezes the processed image into a 1D array.
- A Dropout layer to prevent overfitting by dropping 50% of the neurons randomly.
- A Dense layer with 128 neurons and a ReLU activation function.
- A final Dense later with 36 neurons and a Softmax activation function.

#### Generating Data

We have two methods to generate data:

1. **Running our license plate detection algorithm** on the camera images we took as we drove around the course. We keep restarting the simulation as each restart creates a new set of license plates so that we can obtain all characters needed. Manually label each character by sorting the characters to their own folders and naming them based on their folders.
   - We were able to manually select 632 data where 20% is used for validation.

2. **Generating random license plates**, identical to that of Lab 6, and cropping each character. Use one-hot encoding to label each character.
   - We generated 400 license plates (1600 characters) and 20% is used for validation. This number is selected because we found that it is the sufficient amount of data to train the model while still achieving minimal loss and high accuracy (refer to Appendix B).

The initial plan was to use 36 classes for each letter and number. However, the number on the parking position is using a different font from the license plate. We had the option to either:

- Add 8 more classes for each parking number (1-8) resulting a total of 44 classes.
- Make 2 separate CNN models for reading the license plate and parking position.

<div class="image-gallery">
  <!-- Row 1 -->
  <div class="image-row" style="display: flex; gap: 24px; margin-bottom: 20px;">
    <div class="image-item" style="flex: 1; text-align: center;">
      <img src="/assets/img/projects/traffibot/tb_5_1.png" alt="Generated License Plates" style="max-width: 100%; height: auto;">
    </div>
    <div class="image-item" style="flex: 1; text-align: center;">
      <img src="/assets/img/projects/traffibot/tb_6_1.png" alt="Actual License Plates" style="max-width: 100%; height: auto;">
    </div>
  </div>

  <!-- Row 2 -->
  <div class="image-row" style="display: flex; gap: 24px; margin-bottom: 20px;">
    <div class="image-item" style="flex: 1; text-align: center;">
      <img src="/assets/img/projects/traffibot/tb_5_2.png" alt="Generated License Plates 2" style="max-width: 100%; height: auto;">
    </div>
    <div class="image-item" style="flex: 1; text-align: center;">
      <img src="/assets/img/projects/traffibot/tb_6_2.png" alt="Actual License Plates 2" style="max-width: 100%; height: auto;">
    </div>
  </div>

  <!-- Row 3 -->
  <div class="image-row" style="display: flex; gap: 24px; margin-bottom: 20px;">
    <div class="image-item" style="flex: 1; text-align: center;">
      <img src="/assets/img/projects/traffibot/tb_5_3.png" alt="After Resizing" style="max-width: 100%; height: auto;">
      <p><em>Fig 5: Generated License Plates and After Resizing</em></p>
    </div>
    <div class="image-item" style="flex: 1; text-align: center;">
      <img src="/assets/img/projects/traffibot/tb_6_3.png" alt="Actual License Plates 3" style="max-width: 100%; height: auto;">
      <p><em>Fig 6: Actual License Plates </em></p>
    </div>
  </div>
</div>

After getting sufficient data with all the characters present and cropping each one, the images are resized into one size, are masked (black and white), and one-hot encoded to be trained by the CNN.

#### Training Parameters

```python
LEARNING_RATE = 1e-4
VALIDATION_SPLIT = 0.2 #20% validation, 80% training
conv_model.compile(loss='categorical_crossentropy',
                   optimizer=optimizers.RMSprop(learning_rate=LEARNING_RATE),
                   metrics=['acc'])

# Training the model on generated data
history_conv = conv_model.fit(x_dataset, y_dataset, 
                              validation_split=VALIDATION_SPLIT, 
                              epochs=30)
# Training the model on actual data
history_conv = conv_model.fit(x_dataset, y_dataset, 
                              validation_split=VALIDATION_SPLIT, 
                              epochs=50)
```

---

## Conclusion 📊

### Drive

Overall we can conclude that a finite state machine is an appropriate and doable approach for driving the competition surface. An advantage of this approach will also be that if the competition surface changes but stays with the same characteristics, such as white lines, clear sky, and clear red line, the robot will still be able to traverse the surface.

The main problem with our approach was the lack of preparation for this programming assignment. Under better circumstances we should have constructed a more object oriented state machine that takes in updates from a robot rospy object that preform actions and determines the state of the machine while evaluating the actions that the robot need to take.

### License Plate Recognition

#### Summarize the Performance

Our license plate detection algorithm performed well in Jupyter notebook and can successfully obtain the license plates. We also trained the CNN with the limited data we had available and it showed promising results. However, we encountered difficulties when attempting to load the trained CNN model into the competition platform which prevented us from testing its capabilities. This problem persisted all the way through the competition and we were unable to see it work. We acknowledge that we needed more time to work on the CNN model.

#### Methods We Tried

Initially, we thought using the SIFT algorithm would be a straightforward approach where we use feature mapping, homography and perspective transform to generate the license plate image. However, the algorithm has the tendency to lag and cause delay, thus we moved away from this approach. Iterations are constantly made in our final approach where we experimented with masking, hsv colour thresholds, cropping and contours. This is necessary as the competition environment produces different shades of brightness and contrast due to the angle of the sun from the position of the robot.

#### Areas for Improvement

- **Improving our dataset in training the CNN:** Understanding the key role that data plays in the performance of our CNN model, we would improve our model by enhancing our dataset. This can be done through collecting data that resembles the actual data that our CNN will operate on. Although our generated data is useful, it differs from the real data due to its better resolution, quality and minimal noise. Through consulting with others, we can leverage the "ImageDataGenerator" function from Keras which is a utility for generating batches of augmented data for training deep neural networks that work with image data. It applies various transformations to the input images, such as rotation, shifting, zooming, flipping, and shearing, which increases the size of the training dataset and helps to prevent overfitting. Additionally, we can also allocate more time to acquire more actual data since the data we gathered was insufficient and missing a few characters (refer to Appendix C).

- **Refining our CNN model:** We would also devote more time and effort to fine-tuning our training parameters to achieve the desired level of accuracy and performance of the CNN model.

- We were unable to develop a CNN model for detecting parking positions within the scope of this project and would have implemented it.

The license plate identification posed several challenges and time constrain. With more time, we would have focused on acquiring more datasets to enhance the CNN's performance. We also realize that we should have prioritized training the model over optimizing the license plate identification algorithm. Overall, this project taught us the importance of balancing different aspects of a complex task to achieve the best possible outcome. In the future, we would approach tasks more strategically by taking a step back and gaining a comprehensive understanding of the project's scope before diving into the work. By doing so, we will be able to better identify key tasks and ensure that my time and effort are focused on the right priorities to achieve optimal results.

---

## Appendix 📋

### Appendix A: State Machine Actions

<div class="project-hero-image">
  <img src="/assets/img/projects/traffibot/tb_app_a.png" alt="State Machine Actions" class="project-image">
</div>

### Appendix B: Model Loss and Accuracy from Training the CNN with 1600 Generated Data

<div class="image-gallery">
  <div class="image-item">
    <img src="/assets/img/projects/traffibot/tb_app_b.png" alt="Model Loss" class="project-image">
  </div>
  
  <div class="image-item">
    <img src="/assets/img/projects/traffibot/tb_app_b_2.png" alt="Model Accuracy" class="project-image">
  </div>
</div>

### Appendix C: Histogram of the Data Used for Training

The second histogram reveals some missing characters, indicating that insufficient data was collected for those particular characters. This highlights the need for a more comprehensive and diverse dataset to ensure the CNN model can accurately recognize and classify all characters.

<div class="image-gallery">
  <div class="image-item">
    <img src="/assets/img/projects/traffibot/tb_app_c_1.png" alt="Data Histogram 1" class="project-image">
  </div>
  
  <div class="image-item">
    <img src="/assets/img/projects/traffibot/tb_app_c_2.png" alt="Data Histogram 2" class="project-image">
  </div>
</div>

---

<a href="/" class="back-link">← Back to Home</a> 