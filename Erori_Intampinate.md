#########################################

Erori Intamipinate la instalare:

legacy-install-failure pentru libraria pyaudio
  mesajul errori indica o versiune Microsoft Visual C++ outdated, cu ajutorul Microsoft C++ Build Tools am trecut peste aceasta eroare:

  https://visualstudio.microsoft.com/visual-cpp-build-tools/

  dupa update urmatorul mesaj de eroare: *src/_portaudiomodule.c(29): fatal error C1083:*
  
  dupa putin research pe *Stack Overflow* am descoperit ca versiunea oficiala pyaudio nu are support pentru python 3.7+
  
  pentru versiuni python 3.7+ exista fisiere .whl neoficiale care faciliteaza instalarea pyaudio
  
  https://www.lfd.uci.edu/~gohlke/pythonlibs/#pyaudio
  
  fisierul ales este PyAudio-0.2.11-cp39-cp39-win32.whl, alegerea se face dupa versiunea de python si dupa versiunea mediului de programare (32bit sau 64bit)
  
  pentru ca fisierul sa mearga pe toate subversiunile 3.9 (3.9.1, 3.9.4,...) numele fisierului trebuie modificat astfel: PyAudio-0.2.11-cp39-none-any.whl
  
  instalarea se face folosind comanda de instalare cu trimitere la locatia fisierului.
  
  Nota: pentru fisierul whl a fost redenumita si versiunea in biti, dar aceasta nu face fisierul compatibil cu cealalta versiune, acesta ramane tot pentru 32bit.
  
Dupa realizarea acestor pasi toate librariile au fost instalate cu succes // 15.07.2022

Nota: Librariile numpy si opencv nu au prezentati dificultati deoarece acestea au mai fost folosite in proiecte anterioare.

metoda descrisa anterior nu a dat efect, rezultand in eroarea Could not import the PyAudio C module '_portaudio'. la rularea programului

instalarea librarieri pipwin si folosirea acesteia a adus la instalarea corecta a librariei pyaudio

*py -m pipwin install pyaudio*

#########################################

Erori intampinate dealungul proiectului:

Erori Opencv:

  -Eroare Opencv la implementarea multiprocessing:

    -Functia de scriere a fisierelor video din opencv nu functioneaza corect, rezultand intr-un fisier corupt, daca este folosita in interiorul unu subprocess.
    
    Rezolvare: Folosirea unui Queue pentru a trimite date de la subprocess la procesul main, scrierea videoului in procesul main si inchiderea fisierului in 
    procesul main rezulta la un fisier video necorupt
    
  -Eroare Opencv la conversia in mp4:
  
    -Videoclipul creat in formatul mp4 nu reda video, acesta avand duratia corecta. Salvarea videoului sub extensia .avi si folosind codecul corespunzator
    rezulta intr-un videoclip care reda video
    
    Rezolvare: library-ul moviepy folosit pentru alipirea video si audio nu accepta fisiere .avi, doar .mp4 pentru video
    totodata videoclipul final redat de aceasta are video si audio prezente si corecte. Deci nu trebuie implementat un fix momentan
    
    
Erori pyaudio:

  Pentru a inregistra system sound output este nevoie de Stereo Mix sa fie Enabled, singurul speaker folosit de system sa fie cel de la Realtek(R) Audio
  si sa fie un device connectat prin jack audio la calculator. Pasii au fost detaliati in documentatie.
  In caz contrar, daca stereo mix nu este enabled, sau este inexistent, programul va da defalut la channel 1 dar inregistrarea audio nu va avea sunet.
  
  Alta rezolvare nu a fost gasita momentan.
  
Erori Selenium:

  Erori la pierderea conexiunii/ conexiune lenta
  
  Rezolvare: Exception Handling
  
Erori si dificultati generale:

  -Dificultati in crearea fisierului log:
  
    -Subprocesele create nu au aceleasi permisiuni la fisiere precum procesul main. Pentru a transmite date acestea trebuie sa foloseasca un procedeu precum Queue
    si sa trimita catre procesul main, ca acesta sa noteze log-urile. Redirectionarea outputului din consola catre un fisier folosind sys.stdout nu aduce 
    rezultatul dorit.
    
  Rezolvare: folosind libraria Logging putem extrage loguri personalizate despre functionarea programului, fisierul este creat cu append, acesta trebuie golit manual
    
