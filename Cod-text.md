######################################

#Starea programului dupa weekend // 17.07.2022

#Bullet points:

#-connectare la chrome browser reusita

#-creare fisier video si audio reusita

#-segmentarea programului pe mai multe proccese reusita

#EDIT // 19.07.2022 Versiune finala

#Error Handling adaugat

#Loguri adaugate in fisier si in consola // fisierul log este deschis cu append, acesta nu se va goli

#Automatizare completa

#Vizualizeaza fisierul raw pentu a copia corect codul

#Finishing touches added

######################################

# importing the required packages
import pyautogui
import cv2
import numpy as np
import selenium.common.exceptions
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import keyboard
from multiprocessing import Process, Queue
import pyaudio
import wave
import time
from moviepy.editor import *
from scipy.io import wavfile
import soundfile as sf
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
import logging


# video config
resolution = (1920, 1080)
codec = cv2.VideoWriter_fourcc(*"mp4v")#tfisier salvat ca si mp4 deoarece libraria moviepy nu accepta format video .avi
filename = "Recording.mp4"
fps = 20.0
out = cv2.VideoWriter(filename, codec, fps, resolution)
#onfigurarea video facuta la inceputul fisierului asigura accesul functiilor necesare la date


#sound config
sound  = True
CHUNK = 1024
FORMAT = pyaudio.paInt16
CHANNELS = 2
RATE = 44100
RECORD_SECONDS = 60
p = pyaudio.PyAudio()
#onfigurarea audio facuta la inceputul fisierului asigura accesul functiilor necesare la date


t = time.localtime()# folosit pentru detalii consola si loguri
logging.basicConfig(filename="RecorderLog.log",
                    filemode='a',
                    level=logging.INFO)
logger = logging.getLogger()
logger.setLevel(logging.INFO)
#folosit pentru a loga cu succes datele necesare

def rms_flat(a):  # from matplotlib.mlab
    """
    Return the root mean square of all the elements of *a*, flattened out.
    """
    return np.sqrt(np.mean(np.absolute(a)**2))


def measure_wav_db_level(wavFile):
    """
    Open a wave or raw audio file and perform the following tasks:
    - compute the overall level in db (RMS of data)
    """

    try:
        fs, x = wavfile.read(wavFile)
        LOG_SCALE = 20*np.log10(32767)
    except:
        x, fs = sf.read(wavFile,
                        channels=2, samplerate=44100,
                        format='RAW', subtype='PCM_16')
        LOG_SCALE = 0
    t = (np.array(x)).astype(np.float64)
    # x holds the int16 data, but it's hard to work on int16
    # t holds the float64 conversion
    print(wavFile)
    print(str(fs) + ' Hz')
    print(str(len(t) / fs) + ' s')
    orig_SPL = 20*np.log10(rms_flat(t)) - LOG_SCALE
    print('Sound level:   ' + str(orig_SPL) + ' dBFS\n')
    logger.info('Sound level:   ' + str(orig_SPL) + ' dBFS in fisierul '+wavFile+'\n')
    return str(orig_SPL)



def render(video,audio):
    try:
        videoclip = VideoFileClip(video)
    except OSError:
        print("fisierul video nu este identificat, posibil corupt" +time.strftime("%H:%M:%S", t)+"\n")
        logger.info("fisierul video nu este identificat, posibil corupt" +time.strftime("%H:%M:%S", t)+"\n")
        exit()
    try:
        audioclip = AudioFileClip(audio)
    except OSError:
        print("fisierul audio nu este identificat, posibil corupt" + time.strftime("%H:%M:%S", t) + "\n")
        logger.info("fisierul audio nu este identificat, posibil corupt" + time.strftime("%H:%M:%S", t) + "\n")
        exit()
    new_audioclip = CompositeAudioClip([audioclip])
    videoclip.audio = new_audioclip
    videoclip.write_videofile("Vidfin.mp4")
    videoclip.close()
    audioclip.close()
    new_audioclip.close()
    print("Video final mp4 creat cu succes! " + time.strftime("%H:%M:%S", t))
    logger.info("Video final mp4 creat cu succes! " + time.strftime("%H:%M:%S", t))
#programul se opreste daca unul dintre fisiere lipseste
#fisierele sunt generate pe parcursul rularii, daca acestea lipsesc undeva este o greseala


def sound():
    print("Identificare device index.. "+time.strftime("%H:%M:%S", t)+"\n")
    logger.info("Identificare device index.. "+time.strftime("%H:%M:%S", t)+"\n")
    for i in range(p.get_device_count()):
        dev = p.get_device_info_by_index(i)
        if (dev['name'] == 'Stereo Mix (Realtek(R) Audio)' and dev['hostApi'] == 0):
            dev_index = dev['index'];
            print('dev_index', dev_index)
            print("Index identificat! " + time.strftime("%H:%M:%S", t) + "\n")
            logger.info("Index identificat! " + time.strftime("%H:%M:%S", t) + "\n")

#functia cauta indexul Stereo mix pentru a asigura output audio in fisier
#in cazul in care acesta lipseste, fisierul audio nu va avea sunet
    try:
        stream = p.open(format=FORMAT,
                    channels=CHANNELS,
                    rate=RATE,
                    input=True,
                    input_device_index=dev_index,
                    frames_per_buffer=CHUNK)
    except UnboundLocalError:
        print("Index neidentificat.... stereo mix nu e pornit, default to channel 1" + time.strftime("%H:%M:%S",
                                                                                                     t) + "\n")
        logger.info("Index neidentificat.... stereo mix nu e pornit, default to channel 1" + time.strftime("%H:%M:%S",
                                                                                                     t) + "\n")
        stream = p.open(format=FORMAT,
                        channels=CHANNELS,
                        rate=RATE,
                        input=True,
                        input_device_index=0,
                        frames_per_buffer=CHUNK)
    print("Pornire inregistrare audio "+time.strftime("%H:%M:%S", t)+"\n")
    logger.info("Pornire inregistrare audio "+time.strftime("%H:%M:%S", t)+"\n")

    frames = []
    try:
        for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
            data = stream.read(CHUNK)
            frames.append(data)
            if keyboard.is_pressed('q'):
                break
    except IndexError:
        print("Eroare citire stream audio"+time.strftime("%H:%M:%S", t)+"\n")
        logger.info("Eroare citire stream audio"+time.strftime("%H:%M:%S", t)+"\n")

    print("Inregistrare audio incheiata, audiofile deschis "+time.strftime("%H:%M:%S", t)+"\n")
    logger.info("Inregistrare audio incheiata, audiofile deschis "+time.strftime("%H:%M:%S", t)+"\n")

    wf = wave.open("output1.wav", 'wb')
    wf.setnchannels(CHANNELS)
    wf.setsampwidth(p.get_sample_size(FORMAT))
    wf.setframerate(RATE)
    wf.writeframes(b''.join(frames))
    wf.close()
    print("Audiofile inchis "+time.strftime("%H:%M:%S", t)+"\n")
    logger.info("Audiofile inchis "+time.strftime("%H:%M:%S", t)+"\n")
#scrierea audio nu necesita transfer de date catre procesul main
#aceasta inregistreaza si inchide fisierul cu succes



def cap(a):
    timeout = time.time() + 65
    logger.info("Pornire inregistrare video " + time.strftime("%H:%M:%S", t))
    print("Pornire inregistrare video " + time.strftime("%H:%M:%S", t) )
    #daca fereastra cv2 nu este vizibila, inputul de la tastatura nu se aplica la cv2.waitKey
    #drept pentru care inregistrarea este setata la un timer din program
    while True:
        img = pyautogui.screenshot()
        frame = np.array(img)
        #cv2.imshow('Live', frame)
        a.put(frame)
        #transferul de date creat catre procesul main
        #in procesul main se desfasoara scrierea fisierului video
        if cv2.waitKey(1) == ord('s') or time.time() > timeout:
            a.put(np.array(None))
            break

    print("Inregistrare video incheiata " + time.strftime("%H:%M:%S", t) )
    logger.info("Inregistrare video incheiata " + time.strftime("%H:%M:%S", t) )

    cv2.destroyAllWindows()
    a.close()
    print("Inchidere ferestre cv2 " +time.strftime("%H:%M:%S", t) )
    logger.info("Inchidere ferestre cv2 " +time.strftime("%H:%M:%S", t))



def ytb():
    print("Deschidere fereastra chrome " + time.strftime("%H:%M:%S", t) )
    try:

        options = webdriver.ChromeOptions()
        options.add_experimental_option("detach", True)
        driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)
        driver.get("https://www.youtube.com")
        print("Browser Deschis "+time.strftime("%H:%M:%S", t))
        logger.info("Browser Deschis "+time.strftime("%H:%M:%S", t))
    except selenium.common.exceptions.WebDriverException:
        print("Fara conexiune la Internet" +time.strftime("%H:%M:%S", t))
        logger.info("Fara conexiune la Internet" +time.strftime("%H:%M:%S", t))
        return
    try:
        accept_button = WebDriverWait(driver, 3).until(EC.element_to_be_clickable((By.XPATH,
                                                                                        "/html/body/ytd-app/ytd-consent-bump-v2-lightbox/tp-yt-paper-dialog/div[4]/div[2]/div[6]/div[1]/ytd-button-renderer[2]/a/tp-yt-paper-button")))
        accept_button.click()

    except selenium.common.exceptions.TimeoutException:
        print("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        logger.info("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        return

    time.sleep(2)
    #print("Passed Cookies.. "+time.strftime("%H:%M:%S", t))

    try:
        searchbox = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.XPATH,
                                                                                "/html/body/ytd-app/div[1]/div/ytd-masthead/div[3]/div[2]/ytd-searchbox/form/div[1]/div[1]/input")))
    except selenium.common.exceptions.TimeoutException:
        print("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        logger.info("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        return

    try:
        searchbox.send_keys("2Scratch - Secrets")
        print("Alegere video "+time.strftime("%H:%M:%S", t))
        logger.info("Browser Deschis "+time.strftime("%H:%M:%S", t))
    except selenium.common.exceptions.TimeoutException:
        print("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        logger.info("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        return

    try:
        search_button = WebDriverWait(driver, 20).until(EC.element_to_be_clickable(
            (By.XPATH, "/html/body/ytd-app/div[1]/div/ytd-masthead/div[3]/div[2]/ytd-searchbox/button")))
        search_button.click()
        print("Cautare video "+time.strftime("%H:%M:%S", t))
        logger.info("Cautare video "+time.strftime("%H:%M:%S", t))
    except selenium.common.exceptions.TimeoutException:
        print("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        logger.info("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        return

    try:
        firstvideo = WebDriverWait(driver, 20).until(EC.element_to_be_clickable((By.XPATH,
                                                                                 "/html/body/ytd-app/div[1]/ytd-page-manager/ytd-search/div[1]/ytd-two-column-search-results-renderer/div/ytd-section-list-renderer/div[2]/ytd-item-section-renderer/div[3]/ytd-video-renderer[1]/div[1]/div/div[1]/div/h3/a")))
        firstvideo.click()
    except selenium.common.exceptions.TimeoutException:
        print("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        logger.info("Fara conexiune la Internet" + time.strftime("%H:%M:%S", t))
        return
#daca browserul pierde conexiunea programul va continua pentru a putea verifica restul functionalitatilor
#am ales un wait time de 20s de secunde pentru a evita un timeout pentru conexiunile slabe


#functia main/procesul main care ruleaza aplicatia
if __name__=="__main__":
    x=Queue()
    print("Initializarea programului " + time.strftime("%H:%M:%S", t) )
    logger.info("Initializarea programului " + time.strftime("%H:%M:%S", t))
    p1=Process(target=cap, args=(x,))
    p12=Process(target=sound,args=())
    p2=Process(target=ytb, args=())
    #codul foloseste multiprocess pentru a separa procesele necesare rularii
    #am ales libraria multiprocess in favoarea librariei threading pentru a evita GIL
    #Global Interpreter Lock
    print("Procese create cu succes " + time.strftime("%H:%M:%S", t) )
    logger.info("Procese create cu succes " + time.strftime("%H:%M:%S", t) )
    p1.start()
    p12.start()
    p2.start()
    print("Toate procesele ruleaza " + time.strftime("%H:%M:%S", t) )
    logger.info("Toate procesele ruleaza " + time.strftime("%H:%M:%S", t))
    #crearea unui stream de date de la subprocesul care inregistreaza ecranul
    a=x.get()
    while a.any():
        out.write(cv2.cvtColor(a, cv2.COLOR_BGR2RGB))
        a=x.get()
    #subprocesele au acces limitat la fisiere, acestea depinzand de procesul main pentru scrierea de date cu succes
    #astfel fisierul Recording.mp4 nu va fi corupt la deschidere
    p1.join()
    p12.join()
    p2.join()

    print("Process join.. " + time.strftime("%H:%M:%S", t) )
    logger.info("Process join.. " + time.strftime("%H:%M:%S", t))

    p1.close()
    p2.close()
    p12.close()

    print("Procese incheiate cu succes " + time.strftime("%H:%M:%S", t) )
    logger.info("Procese incheiate cu succes " + time.strftime("%H:%M:%S", t))

    out.release()
    #Inchiderea fisierului Recording.mp4 trebuie facuta dupa incheierea subprocesului responsabil cu captura video
    #nerespectarea acestei ordini duce la o exceptie si coruperea fisierului video
    print("Inchidere videofile " + time.strftime("%H:%M:%S", t) )
    logger.info("Inchidere videofile " + time.strftime("%H:%M:%S", t))

    render("Recording.mp4","Output1.wav")


    measure_wav_db_level("Output1.wav")
    measure_wav_db_level("Vidfin.mp4")
