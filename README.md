# chat-cuma
chat cuma flaite flaiteza artificial
import speech_recognition as sr
from openai import OpenAI
import tkinter as tk
from tkinter import scrolledtext, messagebox

# Configura tu clave de API de OpenAI aquí
client = OpenAI(
  api_key="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"
)

# Inicializa el reconocedor
mic = sr.Recognizer()

def escuchar_y_responder():
    try:
        with sr.Microphone() as source:
            # Ajusta para el ruido ambiental
            mic.adjust_for_ambient_noise(source)
            print("Escuchando... Di algo!")
            status_label.config(text="Escuchando...")
            audio = mic.listen(source, timeout=5)
        
        # Convierte el audio a texto usando Google Speech Recognition
        texto = mic.recognize_google(audio, language='es-ES')  # Asumiendo español; cambia si es necesario
        print(f"Tú dijiste: {texto}")
        
        # Actualiza la interfaz
        output_text.insert(tk.END, f"Tú: {texto}\n")
        output_text.see(tk.END)
        status_label.config(text="Procesando con ChatGPT...")
        
        # Envía el texto directamente como prompt a ChatGPT
        response = client.chat.completions.create(
            model="gpt-3.5-turbo",  # O usa "gpt-4o" si tienes acceso
            messages=[
                {"role": "system", "content": "Eres un asistente útil."},
                {"role": "user", "content": texto}
            ],
            max_tokens=150,
            temperature=0.7
        )
        
        # Imprime la respuesta de ChatGPT
        respuesta = response.choices[0].message.content.strip()
        print(f"ChatGPT: {respuesta}")
        
        # Actualiza la interfaz
        output_text.insert(tk.END, f"ChatGPT: {respuesta}\n\n")
        output_text.see(tk.END)
        status_label.config(text="Listo. Presiona el botón para hablar de nuevo.")
        
    except sr.UnknownValueError:
        messagebox.showerror("Error", "No se pudo entender el audio. Intenta de nuevo.")
        status_label.config(text="Error en reconocimiento. Intenta de nuevo.")
    except sr.RequestError as e:
        messagebox.showerror("Error", f"Error en el servicio de reconocimiento: {e}")
        status_label.config(text="Error en servicio. Revisa conexión.")
    except Exception as e:  # Captura errores de OpenAI u otros
        messagebox.showerror("Error", f"Error: {e}")
        status_label.config(text="Error. Revisa la conexión o clave API.")

# Crear la ventana principal
root = tk.Tk()
root.title("Asistente de Voz con ChatGPT")
root.geometry("600x400")

# Etiqueta de estado
status_label = tk.Label(root, text="Presiona el botón para hablar", font=("Arial", 12))
status_label.pack(pady=10)

# Botón para escuchar
listen_button = tk.Button(root, text="Hablar", command=escuchar_y_responder, font=("Arial", 14), bg="lightblue", padx=20, pady=10)
listen_button.pack(pady=10)

# Área de texto para mostrar la conversación
output_text = scrolledtext.ScrolledText(root, wrap=tk.WORD, width=70, height=20, font=("Arial", 10))
output_text.pack(pady=10, padx=10, fill=tk.BOTH, expand=True)

# Iniciar la interfaz
root.mainloop()
