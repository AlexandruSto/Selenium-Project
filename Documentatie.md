# Selenium-Project

Mediul de programare folosit este PyCharm pentru Windows 10, versiunea de python 3.9.10

Package manager: pip sau pipwin in cazul in care pip nu reuseste instalarea corecta

Browser folosit de catre program: Chrome browser // fara acesta rularea programului nu se poate face

https://www.jetbrains.com/pycharm/download/#section=windows

Librarii instalate pentru proiect:

*Numpy* 

*pyautogui*

*opencv*

*pyaudio*

Instalarea s-a facut folosind pip: py -m pip install "library name"

instalarea librarieri pipwin si folosirea acesteia a adus la instalarea corecta a librariei pyaudio

*py -m pipwin install pyaudio*

alte librarii instalate pe parcursul proiectului:

*multiprocessing*

*wave*

*time*

*keyboard*

*moviepy*

*soundfile*

*logging*

*scipy.io*

################################

Rularea Programului:

Inainte de a rula programul acesta necesita o configuratie audio pentru a capta sunetul videoclipului. Programul necesita ca Stereo Mix sa fie enabled de la
Settings>Sound>Manage Sound Devices. Totodata daca calculatorul are mai multe dispozitive playback inafara Speakers Realtek(R) Audio, acestea trebuie dezactivate de
la Settings>Sound>Manage Sound Devices. De asemena calculatorul necesita un dispozitiv audio connectat la portul Audio Jack.

Tutorial cu detaliat pentru setarile audio necesare.

https://www.tutorialexample.com/python-record-audio-from-computer-speaker-on-win-10-python-tutorial/

Fara aceste setari programul nu va inregistra audio.

Dupa rularea executabilului Cod.py, acesta downloadeaza driverul de chrome si il retine in memoria cache la fiecare rulare, nu este necesara atribuirea unui 
path catre acest driver. Codul va crea 4 fisiere: 
RecorderLog.txt - acesta retine datele despre rularea programului si nivelul de decibeli al fisierelor video; 
Output1.wav - acesta este inregistrarea audio creata; 
Recording.mp4 - este inregistrarea video creata; 
Vidfin.mp4 - este videoul final cu audio si video combinat.
