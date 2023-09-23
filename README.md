import cv2
import smtplib, ssl
import time
from email import encoders
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
smtp_server = 'smtp.yandex.ru'
port = 465
sender = 'awesome.z.d@yandex.com'
receiver = 'awesome.z.d@yandex.com'
password = 'lwxvsvtrzxgszujh'
subject = "testowy e-mail"
threshold = 5000
cap = cv2.VideoCapture(0)
_, bg = cap.read()
bg = cv2.cvtColor(bg, cv2.COLOR_BGR2GRAY)
bg = cv2.GaussianBlur(bg, (21, 21), 0)

while True:
    ret, frame = cap.read()
    cv2.imshow('Camera', frame)
    cv2.imwrite('camera_image.jpg',frame)
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    gray = cv2.GaussianBlur(gray, (21, 21), 0)
    diff = cv2.absdiff(bg, gray)
    thresh = cv2.threshold(diff, 25, 255, cv2.THRESH_BINARY)[1]
    contours, _ = cv2.findContours(thresh.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    motion_detected = False
    for contour in contours:
        if cv2.contourArea(contour) > threshold:
            motion_detected = True
            break
    if motion_detected:
        tresc = """To jest pierwszy e-mail wysłany w pythonie.
        """
        plik = "camera_image.jpg"
        cv2.imwrite('temp.jpg', frame)
        message = MIMEMultipart()
        message["From"] = sender
        message["To"] = receiver
        message["Subject"] = subject
        message.attach(MIMEText(tresc, "plain"))
        cv2.imwrite('temp.jpg', frame)

        with open(plik, "rb") as f:
            zalacznik = MIMEBase("application", "octet-stream")
            zalacznik.set_payload(f.read())

        encoders.encode_base64(zalacznik)

        zalacznik.add_header(
            "Content-Disposition",
            "attachment; filename=" + plik
        )
        message.attach(zalacznik)
        text = message.as_string()

        ssl_context = ssl.create_default_context()
        with smtplib.SMTP_SSL(smtp_server, port, context=ssl_context) as server:
            server.login(sender, password)
            server.sendmail(sender, receiver, text)

    bg = gray

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
