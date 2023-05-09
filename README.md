# Importowanie potrzebnych bibliotek
import cv2
import smtplib, ssl
import time
from email import encoders
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

# Dodawanie potrzebnych do wysyłania maila zmiennych
smtp_server = 'smtp.yandex.ru'
port = 465
sender = 'awesome.z.d@yandex.com'
receiver = 'awesome.z.d@yandex.com'
password = 'lwxvsvtrzxgszujh'
subject = "testowy e-mail"

# Ustawienie czułości detektora ruchu
threshold = 5000

# Tworzenie obiektu kamery
cap = cv2.VideoCapture(0)

# Inicjalizacja wartości tła
_, bg = cap.read()
bg = cv2.cvtColor(bg, cv2.COLOR_BGR2GRAY)
bg = cv2.GaussianBlur(bg, (21, 21), 0)

# Pętla do odczytywania obrazu z kamery
while True:
# Odczytywanie klatki z kamery
    ret, frame = cap.read()
# Wyświetlanie obrazu
    cv2.imshow('Camera', frame)
  
# Zapisanie klatki do pliku
    cv2.imwrite('camera_image.jpg',frame)
# Konwersja klatki na skalę szarości i rozmycie
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = cv2.GaussianBlur(gray, (21, 21), 0)

# Obliczanie różnicy między tłem a klatką
    diff = cv2.absdiff(bg, gray)
    thresh = cv2.threshold(diff, 25, 255, cv2.THRESH_BINARY)[1]

# Znajdowanie konturów obiektów na klatce
    contours, _ = cv2.findContours(thresh.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

# Sprawdzenie, czy ruch jest wystarczająco duży
    motion_detected = False
    for contour in contours:
        if cv2.contourArea(contour) > threshold:
            motion_detected = True
            break
# Wysyłanie maila, jeśli wykryto ruch
    if motion_detected:
        tresc = """To jest pierwszy e-mail wysłany w pythonie.
        """
        plik = "camera_image.jpg"

# Zapisywanie obrazu do pliku tymczasowego
        cv2.imwrite('temp.jpg', frame)
# Tworzenie wiadomości e-mail
        message = MIMEMultipart()
        message["From"] = sender
        message["To"] = receiver
        message["Subject"] = subject
        
# Dodanie treści wiadomości
        message.attach(MIMEText(tresc, "plain"))
        
# Dodanie pliku z obrazem jako załącznik
        cv2.imwrite('temp.jpg', frame)

        with open(plik, "rb") as f:
            zalacznik = MIMEBase("application", "octet-stream")
            zalacznik.set_payload(f.read())

        encoders.encode_base64(zalacznik)

        zalacznik.add_header(
            "Content-Disposition",
            "attachment; filename=" + plik
        )
# Dodanie załącznika do wiadomości e-mail
        message.attach(zalacznik)
        text = message.as_string()

        ssl_context = ssl.create_default_context()
# Połączenie z SMTP i zalogowanie się na konto nadawcy
        with smtplib.SMTP_SSL(smtp_server, port, context=ssl_context) as server:
            server.login(sender, password)
# Wysyłanie wiadomości e-mail
            server.sendmail(sender, receiver, text)

# Aktualizacja tła
    bg = gray

# Czekanie na naciśnięcie klawisza 'q' do zamknięcia okna
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Zwolnienie zasobów
cap.release()
cv2.destroyAllWindows()
