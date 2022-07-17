######################################

#Starea programului dupa weekend // 17.07.2022

#Bullet points:

#-connectare la chrome browser reusita

#-creare fisier video si audio reusita

#-segmentarea programului pe mai multe proccese reusita

######################################

import pyautogui
import cv2
import numpy as np
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.chrome.options import Options
import keyboard
from multiprocessing import Process, Queue
import pyaudio
import wave
import time


# video config
resolution = (1920, 1080)
codec = cv2.VideoWriter_fourcc(*"mp4v")
filename = "Recording.mp4"
fps = 20.0
out = cv2.VideoWriter(filename, codec, fps, resolution)


#sound config
sound  = True
CHUNK = 1024
FORMAT = pyaudio.paInt16
CHANNELS = 2
RATE = 44100
RECORD_SECONDS = 90
WAVE_OUTPUT_FILENAME = "output.wav"
p = pyaudio.PyAudio()


#functie inregistrare sunet
def sound():
    for i in range(p.get_device_count()):
        dev = p.get_device_info_by_index(i)
        if (dev['name'] == 'Stereo Mix (Realtek(R) Audio)' and dev['hostApi'] == 0):
            dev_index = dev['index'];
            print('dev_index', dev_index)

    stream = p.open(format=FORMAT,
                    channels=CHANNELS,
                    rate=RATE,
                    input=True,
                    input_device_index=1,
                    frames_per_buffer=CHUNK)
    print("* recording")
    frames = []
    for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
        data = stream.read(CHUNK)
        frames.append(data)
        if keyboard.is_pressed('q'):  # if key 'q' is pressed
            break
    print("* done recording")
    print("in 5 seconds program will exit")
    wf = wave.open("output1.wav", 'wb')
    wf.setnchannels(CHANNELS)
    wf.setsampwidth(p.get_sample_size(FORMAT))
    wf.setframerate(RATE)
    wf.writeframes(b''.join(frames))
    wf.close()


#fucntie inregistrare video
def cap(a):
    timeout = time.time() + 95
    while True:
        img = pyautogui.screenshot()
        frame = np.array(img)
        #cv2.imshow('Live', frame)
        a.put(frame)
        if cv2.waitKey(1) == ord('s') or time.time() > timeout:
            a.put(np.array(None))
            break
    print("video recording stopped")
    cv2.destroyAllWindows()



#functie navigare web
def ytb():
    try:
        chrome_options = Options()
        chrome_options.add_experimental_option("detach", True)
        driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=chrome_options)
        driver.get("https://www.youtube.com/watch?v=SbJdcaQC7cM&ab_channel=2Scratch")
    except :
        pass



#functia main
#separa executa pe 3 procese
if __name__=="__main__":
    x=Queue()
    y = Queue()
    p1=Process(target=cap, args=(x,))
    p12=Process(target=sound,args=())
    p2=Process(target=ytb, args=())
    p1.start()
    p12.start()
    p2.start()
    a=x.get()
    while a.any():
        out.write(cv2.cvtColor(a, cv2.COLOR_BGR2RGB))
        a=x.get()



    p1.join()
    p12.join()
    p2.join()

    p1.close()
    p2.close()
    p12.close()

out.release()

