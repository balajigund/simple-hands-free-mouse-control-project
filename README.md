# simple-hands-free-mouse-control-project
import cv2
import mediapipe as mp
import pyautogui

# Initialize video capture
cap = cv2.VideoCapture(0)

# Initialize MediaPipe Hands
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(static_image_mode=False, max_num_hands=1,
                       min_detection_confidence=0.5, min_tracking_confidence=0.5)
mp_drawing = mp.solutions.drawing_utils

# Get screen size to map coordinates
screen_width, screen_height = pyautogui.size()

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # Flip the frame for natural interaction
    frame = cv2.flip(frame, 1)

    # Convert to RGB for MediaPipe
    image_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(image_rgb)

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)

            # Get index finger tip landmark
            index_finger_tip = hand_landmarks.landmark[mp_hands.HandLandmark.INDEX_FINGER_TIP]

            # Convert normalized coordinates to screen coordinates
            x = int(index_finger_tip.x * screen_width)
            y = int(index_finger_tip.y * screen_height)

            # Move mouse cursor
            pyautogui.moveTo(x, y)

    # Display output
    cv2.imshow("Mouse Control", frame)

    # Exit on pressing 'q'
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
