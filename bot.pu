import requests
import json
import time

# ================ CONFIG ================
BOT_TOKEN = "8381504704:AAEDIbKrhef3GbzsQU0GHoRbVLlS75mV8-I"
EXTERNAL_API_URL = "https://ftosint.world/api/number?key=free7days&num=9876543210"

TELEGRAM_API = "https://api.telegram.org/bot" + BOT_TOKEN + "/"


# ================ SEND MESSAGE ================
def send_message(chat_id, text, reply_markup=None, parse_mode=None):
    payload = {
        "chat_id": chat_id,
        "text": text
    }

    if reply_markup:
        payload["reply_markup"] = reply_markup

    if parse_mode:
        payload["parse_mode"] = parse_mode

    try:
        response = requests.post(
            TELEGRAM_API + "sendMessage",
            json=payload,
            timeout=15
        )
        response.raise_for_status()
        return response.json()

    except requests.RequestException as error:
        print("Send message error:", error)
        return None


# ================ GET UPDATES ================
def get_updates(offset=None):
    params = {"timeout": 30}

    if offset is not None:
        params["offset"] = offset

    try:
        response = requests.get(
            TELEGRAM_API + "getUpdates",
            params=params,
            timeout=35
        )
        response.raise_for_status()

        data = response.json()

        if data.get("ok"):
            return data.get("result", [])

    except (requests.RequestException, ValueError) as error:
        print("Polling error:", error)

    return []


# ================ REPLY KEYBOARD ================
def create_keyboard():
    return {
        "keyboard": [
            [{"text": "📱 Phone Lookup"}]
        ],
        "resize_keyboard": True
    }


# ================ EXTERNAL API ================
def call_external_api(number):
    if not EXTERNAL_API_URL:
        return {
            "error": "External API URL is not configured."
        }

    try:
        response = requests.get(
            EXTERNAL_API_URL,
            params={"number": number},
            timeout=15
        )
        response.raise_for_status()

        return response.json()

    except requests.RequestException:
        return {"error": "External API request failed."}

    except ValueError:
        return {"error": "External API returned invalid JSON."}


# ================ HANDLE MESSAGES ================
def handle_message(message):
    chat_id = message["chat"]["id"]
    text = message.get("text", "").strip()

    if text == "/start":
        send_message(
            chat_id,
            "👋 Welcome!\n\nChoose an option below.",
            reply_markup=create_keyboard()
        )
        return

    if text == "📱 Phone Lookup":
        send_message(
            chat_id,
            "📞 Send 10 digit mobile number:"
        )
        return

    if not text.isdigit() or len(text) != 10:
        send_message(
            chat_id,
            "❌ Invalid input!\n"
            "Please send exactly 10 numeric digits."
        )
        return

    result = call_external_api(text)

    formatted_json = json.dumps(
        result,
        indent=2,
        ensure_ascii=False
    )

    # Escape HTML-sensitive characters using JSON
    # output and send it inside Telegram's <pre> tag.
    safe_json = (
        formatted_json
        .replace("&", "&amp;")
        .replace("<", "&lt;")
        .replace(">", "&gt;")
    )

    if len(safe_json) > 3500:
        send_message(chat_id, "⚠️ API response is too large.")
        return

    send_message(
        chat_id,
        "<pre>" + safe_json + "</pre>",
        parse_mode="HTML"
    )


# ================ MAIN LOOP ================
def main():
    if not BOT_TOKEN:
        print("Please set your BOT_TOKEN first.")
        return

    offset = None

    print("Bot is running...")

    while True:
        updates = get_updates(offset)

        for update in updates:
            offset = update["update_id"] + 1

            message = update.get("message")

            if not message:
                continue

            try:
                handle_message(message)

            except Exception as error:
                print("Handler error:", error)

        time.sleep(1)


if __name__ == "__main__":
    main()
