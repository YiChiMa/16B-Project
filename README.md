# Fabricating a Voice Controlled Robot Car
##### Collaborators: Isaac Shao, Yichi Ma
##### Fall 2019

## About This Project
In this project, we  built a three-wheeled robot that is responsive to various voice command, as the final project of Electrical Engineering 16B at UC Berkeley. We built the circuit that received the audio signals and programmed the microcontroller to process the signals with PCA. With the closed-loop control feedback scheme, the robot can move according to a specific word we said. This is a lab report produced by our group.

![Overall Design](images/photo.png)

## Front End Circuit
We first connect the microphone circuit to the breadboard. Then we built a voltage regulator on the breadboard and tested its output by multimeter to ensure it is 5V. We also implemented the microphone braising circuit with resistors and op-amps. Next, we calculate the resistors and capacitors to make the high pass frequency fall in the required range (250 Hz to 2500 Hz) and built the band pass filter, using the following formula:
f=1/(2πRC)
The values we chose are R = 200kΩ, C = 0.001μF, and f = 796.2 Hz.
We used oscilloscope to tune the microphone’s gain and built up the skeleton of the car and connect the motor drivers to the breadboard with the encoders.

## Schematic
![](images/schematics.png)

## System Identification
We model the open loop system as following:
d[t+1] - d[t] = Θu[t] + B.
Here, d denotes the position of the car at time step t, u denotes the PWM inputs we controlled. The variables we want to find were Θ and B.
We collect the data points by inputting different PWM.
We also defin v[t] = d[t+1] - d[t]. The system of linear equations can be formed as below:
![](images/equation0.png)
Then, we used least squares to solve for vector x.
![](images/leastSquare.png)

## Control
Closed loop and controller values
We have the model out closed loop system as
![](images/equation2.png)
Where kL and kR are arbitrary constant
We define

d[k] = dL[k] - dR[k]

The car would turn right if d[k] > 0 since dL[k] is greater than dR[k].
Similarly, The car would left right if d[k] < 0 since dR[k] is greater than dL[k].

We express d[k+1] in terms of d[k]:

d[k+1] = dL[k+1] - dR[k+1]
=(1-kL-kR)d[k]

We consider (1-kL-kR) as one dimension closed loop A+BK matrix. We then have to select kL and kR by testing. The final values for kL and kR are

kL = 0.1, kR = 0.6


Turning
We implemented the turning code following this formula:

𝛿[𝑘]=𝑑𝐿[𝑘]−𝑑𝑅[𝑘]

From previous part, we have the open-loop input:
![](images/equation3.png)
We came up with the function of delta reference of car turning by the equations and the geometry picture given by the ipython notebook.
![](images/equation4.png)
Since 𝛿[𝑘]=𝑑𝐿[𝑘]−𝑑𝑅[𝑘], we want to express δ[k]=f(r,v∗,l,k)
After derivation, we get

δ[k] = (lv*t)/r

Where l is the distance between the center of the car to its wheels,
v* is the speed and r is the specified radius,
t as the time step

And then we implemented the turning program in Energia and adjusted the value of running time of turning left and right to be both 5000ms to make the car perform a perfect 90-degree turn.

## Principle Component Analysis
We first uploaded the test code to the launchpad to check if the microphone was working correctly. And then we uploaded the recording code to the launchpad to record for 4 groups of each word being repeated for 30 times. We filled in the blank in the ipython notebook with the path and name of the recording files that had interpreted audio to excels sheets of data. After completing these parts, we generated the input matrix of the data and implemented the code to perform SVD, plot the sigma values, and project on to the principal components.
```
processed_A = np.vstack((word1_processed_train, word2_processed_train, word3_processed_train, word4_processed_train))
mean_vec = np.mean(processed_A, axis = 0)
demeaned_A = processed_A - mean_vec
Taking SVD of matrix A: U, S, Vt = np.linalg.svd(demeaned_A)
```
We chose a new basis to project the principal components onto the new basis.
Next, we clustered the data points by implementing the centroids matrix.
```
centroids = [
        np.mean(clustered_data[:num_samples_train], axis=0),
        np.mean(clustered_data[num_samples_train:2*num_samples_train], axis=0),
        np.mean(clustered_data[2*num_samples_train:3*num_samples_train], axis=0),
        np.mean(clustered_data[3*num_samples_train:4*num_samples_train], axis=0),
    ]
```
Finally, we constructed PCA matrix with test data, computed projected mean vector and projected test data the same basis to test if the accuracy of each word reached 80% after classification. Due to this requirement, we record other words to substitute the words recorded before that could not be recognized by the program. And our words transferred from “sing, basketball, rap, music” to “basketball, rap, Sahai, Hilfinger.” “Rap” is hard to catch the syllable of “p” so that it could not be distinguished with “sing” by the car, so we substitute “sing” with “Sahai.” However, it was also hard for the car to tell the differences between “Sahai” and “music” because the stress on the syllables are similar, so we changed “music” to “hilfinger”, which works great in the end.

## Future Improvements
One of the disadvantages of the car is that the words need to different in length and stress of syllables. Plus, the pronunciation of the commands needed to be close to the samples that had been recorded. We think there should be a better way to make the car recognize each word precisely instead of only depends on the length of syllables of words. Most voice assistants on mobile phones and websites had already achieved this goal, like Siri and Cortana. In order to work as the user asked, machines must be able to interpret all kinds of voice commands precisely first. If we have a chance to improve the car, the classification method would be the first to optimize.

Another aspect that needs to be fixed is that the skeleton of the car. In the project, we found that the wheels of the car went loose when we tested it. We think it may be a good idea to improve the connection of the wheels to the car.
