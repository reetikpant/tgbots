import cv2
import telebot
import os
import tempfile

# Replace with your bot token
BOT_TOKEN = 'tokenhere'
bot = telebot.TeleBot(BOT_TOKEN)

# Global variables for watermark settings
watermark_settings = {
    'position': 'bottom_right',  # default position
    'bg': 'light',  # default background
    'text': 'Your Watermark Text Here',  # default watermark text
    'font': 'plain',  # default font style
    'font_scale': 0.5  # default font size
}

# Map of font styles
font_styles = {
    'plain': cv2.FONT_HERSHEY_PLAIN,
    'simplex': cv2.FONT_HERSHEY_SIMPLEX,
    'duplex': cv2.FONT_HERSHEY_DUPLEX,
    'complex': cv2.FONT_HERSHEY_COMPLEX,
    'complex_small': cv2.FONT_HERSHEY_COMPLEX_SMALL,
    'triplex': cv2.FONT_HERSHEY_TRIPLEX,
    'script': cv2.FONT_HERSHEY_SCRIPT_SIMPLEX,
    'script_complex': cv2.FONT_HERSHEY_SCRIPT_COMPLEX
}

# Watermarking function
def add_text_watermark(input_video, output_video, text, position='bottom_right', bg='light', font=cv2.FONT_HERSHEY_PLAIN, font_scale=0.5, color=(255, 255, 255), thickness=1):
    cap = cv2.VideoCapture(input_video)
    
    # Get original video properties
    frame_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    frame_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
    fps = cap.get(cv2.CAP_PROP_FPS)

    # Define the codec and create VideoWriter object
    fourcc = cv2.VideoWriter_fourcc(*'mp4v')  # 'mp4v' is a common codec for preserving quality
    out = cv2.VideoWriter(output_video, fourcc, fps, (frame_width, frame_height), isColor=True)

    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        
        text_size = cv2.getTextSize(text, font, font_scale, thickness)[0]
        text_width, text_height = text_size
        
        # Calculate the position for the watermark text
        if position == 'top_left':
            x_offset, y_offset = 10, text_height + 10
        elif position == 'bottom_left':
            x_offset, y_offset = 10, frame_height - 10
        elif position == 'top_right':
            x_offset, y_offset = frame_width - text_width - 10, text_height + 10
        elif position == 'bottom_right':
            x_offset, y_offset = frame_width - text_width - 10, frame_height - 10
        elif position == 'top_middle':
            x_offset, y_offset = (frame_width - text_width) // 2, text_height + 10
        elif position == 'bottom_middle':
            x_offset, y_offset = (frame_width - text_width) // 2, frame_height - 10
        else:
            x_offset, y_offset = frame_width - text_width - 10, frame_height - 10

        # Add background for text if needed
        if bg == 'dark':
            bg_color = (0, 0, 0)
            cv2.rectangle(frame, (x_offset - 5, y_offset - text_height - 5), (x_offset + text_width + 5, y_offset + 5), bg_color, -1)
            cv2.putText(frame, text, (x_offset, y_offset), font, font_scale, color, thickness)
        else:
            cv2.putText(frame, text, (x_offset, y_offset), font, font_scale, color, thickness)
        
        out.write(frame)
        
    cap.release()
    out.release()

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    commands = (
        "/start - Start the bot\n"
        "/help - Show available commands\n"
        "/set_watermark - Set watermark options\n"
        "/show_watermark - Show current watermark settings\n"
        "/set_text - Set watermark text\n"
        "/set_font - Set watermark font style\n"
        "/set_font_size - Set watermark font size"
    )
    bot.reply_to(message, f"Available commands:\n{commands}")

@bot.message_handler(commands=['set_watermark'])
def set_watermark(message):
    markup = telebot.types.ReplyKeyboardMarkup(row_width=2)
    markup.add(
        telebot.types.KeyboardButton('Position: Top Left'),
        telebot.types.KeyboardButton('Position: Bottom Left'),
        telebot.types.KeyboardButton('Position: Top Right'),
        telebot.types.KeyboardButton('Position: Bottom Right'),
        telebot.types.KeyboardButton('Position: Top Middle'),
        telebot.types.KeyboardButton('Position: Bottom Middle'),
        telebot.types.KeyboardButton('Background: Light'),
        telebot.types.KeyboardButton('Background: Dark')
    )
    bot.reply_to(message, "Choose watermark settings:", reply_markup=markup)

@bot.message_handler(commands=['show_watermark'])
def show_watermark(message):
    position = watermark_settings.get('position', 'bottom_right')
    bg = watermark_settings.get('bg', 'light')
    text = watermark_settings.get('text', 'Your Watermark Text Here')
    font = watermark_settings.get('font', 'plain').title()
    font_scale = watermark_settings.get('font_scale', 0.5)
    bot.reply_to(message, f"Current watermark settings:\nPosition: {position}\nBackground: {bg}\nText: {text}\nFont: {font}\nFont Size: {font_scale}")

@bot.message_handler(commands=['set_text'])
def set_text(message):
    msg = bot.reply_to(message, "Please send me the new watermark text.")
    bot.register_next_step_handler(msg, process_text)

def process_text(message):
    if message.text:
        watermark_settings['text'] = message.text
        bot.reply_to(message, f"Watermark text updated to: {message.text}")
    else:
        bot.reply_to(message, "Invalid text. Please try again.")

@bot.message_handler(commands=['set_font'])
def set_font(message):
    markup = telebot.types.ReplyKeyboardMarkup(row_width=2)
    markup.add(
        telebot.types.KeyboardButton('Font: Plain'),
        telebot.types.KeyboardButton('Font: Simplex'),
        telebot.types.KeyboardButton('Font: Duplex'),
        telebot.types.KeyboardButton('Font: Complex'),
        telebot.types.KeyboardButton('Font: Complex Small'),
        telebot.types.KeyboardButton('Font: Triplex'),
        telebot.types.KeyboardButton('Font: Script'),
        telebot.types.KeyboardButton('Font: Script Complex')
    )
    bot.reply_to(message, "Choose font style:", reply_markup=markup)

@bot.message_handler(commands=['set_font_size'])
def set_font_size(message):
    msg = bot.reply_to(message, "Please send me the new font size (e.g., 0.5, 1.0, 1.5).")
    bot.register_next_step_handler(msg, process_font_size)

def process_font_size(message):
    try:
        font_scale = float(message.text)
        if font_scale > 0:
            watermark_settings['font_scale'] = font_scale
            bot.reply_to(message, f"Font size updated to: {font_scale}")
        else:
            bot.reply_to(message, "Font size must be greater than 0. Please try again.")
    except ValueError:
        bot.reply_to(message, "Invalid font size. Please enter a valid number.")

@bot.message_handler(func=lambda msg: msg.text.startswith('Position:'))
def set_position(message):
    position = message.text.replace('Position: ', '').lower().replace(' ', '_')
    if position in ['top_left', 'bottom_left', 'top_right', 'bottom_right', 'top_middle', 'bottom_middle']:
        watermark_settings['position'] = position
        bot.reply_to(message, f"Watermark position set to {position.replace('_', ' ').title()}.")
    else:
        bot.reply_to(message, "Invalid position option. Please choose again.")

@bot.message_handler(func=lambda msg: msg.text.startswith('Background:'))
def set_background(message):
    bg = message.text.replace('Background: ', '').lower()
    if bg in ['light', 'dark']:
        watermark_settings['bg'] = bg
        bot.reply_to(message, f"Watermark background set to {bg.title()}.")
    else:
        bot.reply_to(message, "Invalid background option. Please choose again.")

@bot.message_handler(func=lambda msg: msg.text.startswith('Font:'))
def set_font_style(message):
    font_name = message.text.replace('Font: ', '').lower().replace(' ', '_')
    if font_name in font_styles:
        watermark_settings['font'] = font_name
        bot.reply_to(message, f"Font style set to {font_name.replace('_', ' ').title()}.")
    else:
        bot.reply_to(message, "Invalid font style. Please choose again.")

@bot.message_handler(content_types=['video'])
def handle_video(message):
    # Download the video to a temporary file
    file_info = bot.get_file(message.video.file_id)
    downloaded_file = bot.download_file(file_info.file_path)

    # Create a temporary file for the downloaded video
    with tempfile.NamedTemporaryFile(delete=False, suffix='.mp4') as input_temp_file:
        input_temp_file.write(downloaded_file)
        input_temp_file_path = input_temp_file.name

    # Create a temporary file for the watermarked video
    with tempfile.NamedTemporaryFile(delete=False, suffix='.mp4') as output_temp_file:
        output_temp_file_path = output_temp_file.name

    # Apply watermark to the video
    text = watermark_settings.get('text', 'Your Watermark Text Here')
    font_style = watermark_settings.get('font', 'plain')
    font = font_styles.get(font_style, cv2.FONT_HERSHEY_PLAIN)
    font_scale = watermark_settings.get('font_scale', 0.5)
    add_text_watermark(input_temp_file_path, output_temp_file_path, text,
                       position=watermark_settings.get('position', 'bottom_right'),
                       bg=watermark_settings.get('bg', 'light'),
                       font=font,
                       font_scale=font_scale)

    # Send the watermarked video back to the user
    with open(output_temp_file_path, 'rb') as video_file:
        bot.send_video(message.chat.id, video_file, caption="Here's your watermarked video!")

    # Clean up temporary files
    os.remove(input_temp_file_path)
    os.remove(output_temp_file_path)

bot.infinity_polling()







'''
import cv2
import telebot
import os
import tempfile

# Replace with your bot token
BOT_TOKEN = '7213897353:AAFeGFvX7JWEonHpe9Q5y86wpMVi4jP374g'
bot = telebot.TeleBot(BOT_TOKEN)

# Global variables for watermark settings
watermark_settings = {
    'position': 'bottom_right',  # default position
    'bg': 'light'  # default background
}

# Watermarking function
def add_text_watermark(input_video, output_video, text, position='bottom_right', bg='light', font=cv2.FONT_HERSHEY_PLAIN, font_scale=0.5, color=(255, 255, 255), thickness=1):
    cap = cv2.VideoCapture(input_video)
    
    frame_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
    frame_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
    fps = int(cap.get(cv2.CAP_PROP_FPS))
    
    fourcc = cv2.VideoWriter_fourcc(*'H264')  # Use H.264 codec
    out = cv2.VideoWriter(output_video, fourcc, fps, (frame_width, frame_height), isColor=True)
    
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        
        text_size = cv2.getTextSize(text, font, font_scale, thickness)[0]
        text_width, text_height = text_size
        
        if position == 'top_left':
            x_offset, y_offset = 10, text_height + 10
        elif position == 'bottom_left':
            x_offset, y_offset = 10, frame_height - 10
        elif position == 'top_right':
            x_offset, y_offset = frame_width - text_width - 10, text_height + 10
        elif position == 'bottom_right':
            x_offset, y_offset = frame_width - text_width - 10, frame_height - 10
        elif position == 'top_middle':
            x_offset, y_offset = (frame_width - text_width) // 2, text_height + 10
        elif position == 'bottom_middle':
            x_offset, y_offset = (frame_width - text_width) // 2, frame_height - 10
        else:
            x_offset, y_offset = frame_width - text_width - 10, frame_height - 10

        if bg == 'dark':
            bg_color = (0, 0, 0)
            cv2.rectangle(frame, (x_offset - 5, y_offset - text_height - 5), (x_offset + text_width + 5, y_offset + 5), bg_color, -1)
            cv2.putText(frame, text, (x_offset, y_offset), font, font_scale, color, thickness)
        else:
            cv2.putText(frame, text, (x_offset, y_offset), font, font_scale, color, thickness)
        
        out.write(frame)
        
    cap.release()
    out.release()

@bot.message_handler(commands=['start', 'help'])
def send_welcome(message):
    commands = (
        "/start - Start the bot\n"
        "/help - Show available commands\n"
        "/set_watermark - Set watermark options\n"
        "/show_watermark - Show current watermark settings"
    )
    bot.reply_to(message, f"Available commands:\n{commands}")

@bot.message_handler(commands=['set_watermark'])
def set_watermark(message):
    markup = telebot.types.ReplyKeyboardMarkup(row_width=2)
    markup.add(
        telebot.types.KeyboardButton('Position: Top Left'),
        telebot.types.KeyboardButton('Position: Bottom Left'),
        telebot.types.KeyboardButton('Position: Top Right'),
        telebot.types.KeyboardButton('Position: Bottom Right'),
        telebot.types.KeyboardButton('Position: Top Middle'),
        telebot.types.KeyboardButton('Position: Bottom Middle'),
        telebot.types.KeyboardButton('Background: Light'),
        telebot.types.KeyboardButton('Background: Dark')
    )
    bot.reply_to(message, "Choose watermark settings:", reply_markup=markup)

@bot.message_handler(commands=['show_watermark'])
def show_watermark(message):
    position = watermark_settings.get('position', 'bottom_right')
    bg = watermark_settings.get('bg', 'light')
    bot.reply_to(message, f"Current watermark settings:\nPosition: {position}\nBackground: {bg}")

@bot.message_handler(func=lambda msg: msg.text.startswith('Position:'))
def set_position(message):
    position = message.text.replace('Position: ', '').lower().replace(' ', '_')
    if position in ['top_left', 'bottom_left', 'top_right', 'bottom_right', 'top_middle', 'bottom_middle']:
        watermark_settings['position'] = position
        bot.reply_to(message, f"Watermark position set to {position.replace('_', ' ').title()}.")
    else:
        bot.reply_to(message, "Invalid position option. Please choose again.")

@bot.message_handler(func=lambda msg: msg.text.startswith('Background:'))
def set_background(message):
    bg = message.text.replace('Background: ', '').lower()
    if bg in ['light', 'dark']:
        watermark_settings['bg'] = bg
        bot.reply_to(message, f"Watermark background set to {bg.title()}.")
    else:
        bot.reply_to(message, "Invalid background option. Please choose again.")

@bot.message_handler(content_types=['video'])
def handle_video(message):
    # Download the video to a temporary file
    file_info = bot.get_file(message.video.file_id)
    downloaded_file = bot.download_file(file_info.file_path)

    # Create a temporary file for the downloaded video
    with tempfile.NamedTemporaryFile(delete=False, suffix='.mp4') as input_temp_file:
        input_temp_file.write(downloaded_file)
        input_temp_file_path = input_temp_file.name

    # Create a temporary file for the watermarked video
    with tempfile.NamedTemporaryFile(delete=False, suffix='.mp4') as output_temp_file:
        output_temp_file_path = output_temp_file.name

    # Apply watermark to the video
    text = "Your Watermark Text Here"
    add_text_watermark(input_temp_file_path, output_temp_file_path, text,
                       position=watermark_settings.get('position', 'bottom_right'),
                       bg=watermark_settings.get('bg', 'light'))

    # Send the watermarked video back to the user
    with open(output_temp_file_path, 'rb') as video_file:
        bot.send_video(message.chat.id, video_file, caption="Here's your watermarked video!")

    # Clean up temporary files
    os.remove(input_temp_file_path)
    os.remove(output_temp_file_path)

bot.infinity_polling()
'''
