# Selenium-Project

Mediul de programare folosit este PyCharm pentru Windows 10, versiunea de python 3.9.10

https://www.jetbrains.com/pycharm/download/#section=windows

Librarii instalate pentru proiect:

*Numpy* 

*pyautogui*

*opencv*

*pyaudio*

Instalarea s-a facut folosind pip: py -m pip install "library name"

Erori Intamipinate:

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


EDIT:

metoda descrisa anterior nu a dat efect, rezultand in eroarea Could not import the PyAudio C module '_portaudio'. la rularea programului

instalarea librarieri pipwin si folosirea acesteia a adus la instalarea corecta a librariei pyaudio

*py -m pipwin install pyaudio*

alte librarii instalate pe parcursul proiectului:

*multiprocessing*

*wave*

*time*

*keyboard*

*moviepy*
