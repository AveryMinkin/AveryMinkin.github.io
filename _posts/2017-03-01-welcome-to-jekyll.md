---
title: "Commonwealth Honors Thesis"
layout: post
---

Summary and updates on my thesis: exploring the viability of motion tracking and feedback during nursing procedures. This work is being done in collaboration with the [Mechatronics and Robotics Research Lab][MRRL] at UMass Amherst. 
Committee Chair: Professor Frank Sup


Summarize work here. Add videos, links to files, updates.

Hand Tracking Script using Mediapipe:

{% highlight ruby %}
    import cv2
    import mediapipe as mp
    import csv
    import os
    mp_drawing = mp.solutions.drawing_utils
    mp_drawing_styles = mp.solutions.drawing_styles
    mp_hands = mp.solutions.hands
    # For webcam input:
    cap = cv2.VideoCapture(0)
    #cap = cv2.VideoCapture("delt inject.mp4")
    #cap =cv2.VideoCapture('Intramuscular.mp4')

    def getHand(image):
        size = 7
        data = [None for i in range(size)]
        with mp_hands.Hands(
            model_complexity=0,
            min_detection_confidence=0.3,
            max_num_hands=5,
            min_tracking_confidence=0.3) as hands:
    # Draw the hand annotations on the image.

            image.flags.writeable = True
            image = cv2.cvtColor(image, cv2.COLOR_RGB2BGR)
            results = hands.process(image)
            if results.multi_hand_landmarks:
                from google.protobuf.json_format import MessageToDict
                for idx, hand_handedness in enumerate(results.multi_handedness):
                    handedness_dict = MessageToDict(hand_handedness)

                if handedness_dict['classification'][0]['label'] == 'Right':
                    print('left')
                    for hand_landmarks in results.multi_hand_landmarks:
                        data.insert(0, hand_landmarks.landmark[0].x)
                        data.insert(1, hand_landmarks.landmark[0].y)
                        data.insert(2, hand_landmarks.landmark[0].z)
                        mp_drawing.draw_landmarks(
                            image,
                            hand_landmarks,
                            mp_hands.HAND_CONNECTIONS,
                            mp_drawing_styles.get_default_hand_landmarks_style(),
                            mp_drawing_styles.get_default_hand_connections_style())
                else:
                    print('right')
                    for hand_landmarks in results.multi_hand_landmarks:
                        data.insert(3, hand_landmarks.landmark[0].x)
                        data.insert(4, hand_landmarks.landmark[0].y)
                        data.insert(5, hand_landmarks.landmark[0].z)
                        mp_drawing.draw_landmarks(
                            image,
                            hand_landmarks,
                            mp_hands.HAND_CONNECTIONS,
                            mp_drawing_styles.get_default_hand_landmarks_style(),
                            mp_drawing_styles.get_default_hand_connections_style())
        return image, data

    with open(os.path.join(os.getcwd(), "cat.csv"), 'w') as f:  
        writer = csv.writer(f)
        writer.writerow(['t', 'L_x', 'L_y', 'L_z', 'R_x', 'R_y', 'R_z', ' ', 'S_x', 'S_y', 'S_z', 'E_x', 'E_y', 'E_z'])
        #while cap.isOpened():
        #success, image = cap.read()

    # To improve performance, optionally mark the image as not writeable to
    # pass by reference.
    t = 0
    while cap.isOpened():
        ret, img = cap.read()
        image, data = getHand(img)
        data = [t,] + data
        t+=1
    # image.flags.writeable = False
    # image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
        with open(os.path.join(os.getcwd(), "cat.csv"), 'a',  newline='') as f:
            writer = csv.writer(f)
            writer.writerow(data)
        cv2.imshow('MediaPipe Hands', cv2.flip(image, 1))
        if cv2.waitKey(5) & 0xFF == 27:
            break

    #if not success:
           # print("Ignoring empty camera frame.")
        # If loading a video, use 'break' instead of 'continue'.    
    cap.release()

{% endhighlight %}


[MRRL]: https://www.umass.edu/robotics/mrrl

