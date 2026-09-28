import RPi.GPIO as GPIO
import time
import requests

# GPIO Pins Configuration
SWITCH_PIN = 17            # Push Button Pin
BUZZER_PIN = 27            # Buzzer Pin
LIGHT_PINS = [22, 23, 24]   # Relay Pins for Lights

# Mobile Notification API Key (Pushbullet)
PUSHBULLET_API_KEY = "YOUR_PUSHBULLET_API_KEY_HERE"

def setup():
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(SWITCH_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)
    GPIO.setup(BUZZER_PIN, GPIO.OUT)
    
    for pin in LIGHT_PINS:
        GPIO.setup(pin, GPIO.OUT)
        GPIO.output(pin, GPIO.HIGH)  # Turn ON lights initially

def send_mobile_notification(title, body):
    url = "https://api.pushbullet.com/v2/pushes"
    headers = {
        "Access-Token": PUSHBULLET_API_KEY,
        "Content-Type": "application/json"
    }
    data = {
        "type": "note",
        "title": title,
        "body": body
    }
    try:
        requests.post(url, json=data, headers=headers, timeout=5)
    except Exception as e:
        print(f"Error sending notification: {e}")

def turn_off_lights():
    for pin in LIGHT_PINS:
        GPIO.output(pin, GPIO.LOW)  # Turn OFF lights

def trigger_alarm():
    for _ in range(5):  # Beep 5 times
        GPIO.output(BUZZER_PIN, GPIO.HIGH)
        time.sleep(0.5)
        GPIO.output(BUZZER_PIN, GPIO.LOW)
        time.sleep(0.5)

def main():
    setup()
    try:
        while True:
            button_state = GPIO.input(SWITCH_PIN)
            if button_state == GPIO.LOW:  # Switch pressed
                turn_off_lights()
                trigger_alarm()
                send_mobile_notification(
                    "Security Alert", 
                    "Main switch pressed. Lights turned OFF and alarm activated."
                )
                time.sleep(2)  # Debounce delay
                
            time.sleep(0.1)

    except KeyboardInterrupt:
        pass
    finally:
        GPIO.cleanup()

if __name__ == "__main__":
    main()
