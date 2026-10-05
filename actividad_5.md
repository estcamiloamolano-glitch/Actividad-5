# Control y Simulación en Tiempo Real de un Brazo Robótico (ESP32 + PyBullet)

Este repositorio contiene la implementación completa de un sistema cibernético de control en bucle abierto para un brazo robótico articulado equipado con un efector final de tipo pinza (gripper). 

El sistema utiliza un microcontrolador **ESP32** para capturar señales analógicas continuas desde tres potenciómetros físicos, procesarlas mediante su convertidor analógico-digital (ADC) de 12 bits, y transmitirlas a través de una interfaz de comunicación serie en tiempo real (**UART / USB**) hacia un entorno de simulación física multicuerpo desarrollado en **Python** utilizando el motor **PyBullet**.

---

## 🎥 Demostración en Video y Fotografía

### Demostración en Video del Montaje en Funcionamiento
Below you can watch the physical assembly interacting in real-time with the 3D digital twin:

<!-- REEMPLAZA EL ENLACE DE ABAJO CON TU VIDEO SUBIDO A GITHUB O YOUTUBE -->
<p align="center">
  <a href="[https://github.com/TU_USUARIO/Brazo_URDF/assets/demo_video.mp4](https://youtube.com/shorts/Xws34grkwh4?si=MOjrPoyY9AP0RlI4)">
    <img width="899" height="1599" alt="image" src="https://github.com/user-attachments/assets/8a01aa81-2f02-4957-a462-f7752f9c003b" />
  </a>
  <br>
  
  <em>Figura 1: Módulo físico ESP32 con potenciómetros y simulador PyBullet operando simultáneamente. Haz clic en la imagen para ver el video.</em>
</p>

---

## 🛠️ Descripción Detallada del Trabajo Realizado

El desarrollo de este proyecto se estructuró en cuatro fases principales de ingeniería:

### 1. Modelado Cinemático y Físico (URDF)
Se diseñó la estructura del robot utilizando el estándar **URDF (Unified Robot Description Format)** en formato XML (`brazo.urdf`). 
* **Base (`base_link`)**: Estructura cilíndrica fija servida como anclaje inercial.
* **Articulación 1 (`joint_1`)**: Articulación rotacional (revoluta) que conecta la base con el primer eslabón, permitiendo rotación horizontal sobre el eje Z en un rango de -2.5 a +2.5 radianes.
* **Articulación 2 (`joint_2`)**: Articulación rotacional que actúa como hombro, permitiendo elevación sobre el eje Y en un rango de -2.0 a +2.0 radianes.
* **Efector Final / Pinza (`joint_dedo_izq` y `joint_dedo_der`)**: Articulaciones prismáticas (traslacionales) simétricas que desplazan los dedos de la pinza de 0 a 0.05 metros (5 centímetros).

### 2. Desarrollo del Firmware C++ para ESP32 (PlatformIO)
Se programó el microcontrolador ESP32 en C++ utilizando el marco de trabajo de Arduino bajo PlatformIO:
* **Lectura ADC**: Se configuró la resolución del ADC1 a 12 bits (rangos digitales de 0 a 4095).
* **Filtrado Digital**: Se implementó un filtro de promediado de 5 muestras consecutivas por canal para atenuar el ruido eléctrico variante causado por la fricción interna de los potenciómetros.
* **Mapeo Físico**: Se desarrollaron funciones de escalado lineal para convertir la lectura digital (0 a 4095) a los rangos físicos requeridos por el modelo URDF (radianes para ejes rotacionales y metros para ejes prismáticos).
* **Protocolo de Comunicación**: Transmisión periódica a una tasa de refresco de 50 Hz (20 milisegundos) mediante una cadena delimitada por comas (CSV) terminada en salto de línea: `q1,q2,q_finger\n`.

### 3. Desarrollo del Script de Control en Python (PyBullet)
Se construyó el motor de simulación y el puente de comunicación en Python (`main.py`):
* **Gestión Serial**: Apertura de canal USB a 115200 baudios usando `pyserial`.
* **Control de Latencia**: Implementación de una rutina de purga de búfer que descarta paquetes antiguos acumulados si la cola de entrada supera los 64 bytes, garantizando respuesta instantánea sin retardo acumulativo.
* **Asignación Cinemática**: Mapeo automático de nombres de articulaciones del URDF hacia los índices internos de PyBullet y aplicación de control de posición mediante `p.setJointMotorControl2`.

---

## 🔌 Esquema de Conexiones de Hardware

### Tabla de Pines y Rangos de Operación

| Componente | Pin ESP32 | Voltaje Alimentación | Variable Mapeada | Rango Min / Max |
| :--- | :--- | :--- | :--- | :--- |
| **Potenciómetro 1 (Cintura)** | GPIO 34 | 3.3V (ADC1) | `joint_1` (Rad) | -2.5 rad a +2.5 rad |
| **Potenciómetro 2 (Hombro)** | GPIO 35 | 3.3V (ADC1) | `joint_2` (Rad) | -2.0 rad a +2.0 rad |
| **Potenciómetro 3 (Pinza)** | GPIO 32 | 3.3V (ADC1) | `joint_dedo_izq` / `der` | 0.0 m a 0.05 m |

> ⚠️ **IMPORTANTE:** Los potenciómetros se conectan exclusivamente a la línea de 3.3V de la ESP32. Conectarlos a 5V superará la tolerancia de entrada del puerto y quemará los canales del microcontrolador.

---

## 🗂️ Estructura del Repositorio

```text
Brazo_URDF/
├── assets/
│   ├── setup.jpg           # Fotografía del montaje físico y simulación
│   └── demo_video.mp4      # Video de demostración del sistema
├── .venv/                  # Entorno virtual de Python (excluido en git)
├── brazo.urdf               # Descripción geométrica y física del brazo robótico
├── main.py                 # Código principal de control y PyBullet
├── README.md               # Documentación general del repositorio
└── firmware_esp32/         # Proyecto C++ PlatformIO
    ├── platformio.ini      # Configuración de tarjeta y entorno de compilación
    └── src/
        └── main.cpp        # Firmware C++ para la ESP32
```

---

## 🚀 Guía de Instalación y Ejecución Paso a Paso

### Prerequisitos
* Visual Studio Code instalado.
* Python versión 3.10 o 3.11 instalado.
* Extensión PlatformIO IDE instalada en VS Code.

### Paso 1: Clonar el Repositorio
```powershell
git clone https://github.com/TU_USUARIO/Brazo_URDF.git
cd Brazo_URDF
```

### Paso 2: Crear y Activar el Entorno Virtual de Python
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### Paso 3: Instalar las Librerías de Python
```powershell
pip install pybullet pyserial
```

### Paso 4: Cargar el Firmware a la ESP32
1. Conecta la tarjeta ESP32 a la PC mediante cable USB.
2. Abre la terminal de VS Code y navega a la carpeta del firmware:
   ```powershell
   cd firmware_esp32
   ```
3. Ejecuta el comando de compilación y subida:
   ```powershell
   & "C:\Users\TU_USUARIO\.platformio\penv\Scripts\pio.exe" run --target upload
   ```
4. Regresa a la carpeta principal:
   ```powershell
   cd ..
   ```

### Paso 5: Iniciar la Simulación 3D
Asegúrate de que la variable `PORT` dentro de `main.py` coincida con el puerto COM de tu ESP32 (por ejemplo, `'COM3'`) y ejecuta:
```powershell
python main.py
```

---

## 🔧 Solución de Problemas Frecuentes

1. **Error `pyserial` o Puerto Ocupado:**
   * Cierra cualquier Monitor Serie abierto en VS Code, Arduino IDE o PuTTY antes de iniciar `main.py`.
2. **El robot responde con retraso:**
   * Verifica que en `main.py` esté activa la instrucción de limpieza de búfer `ser.in_waiting > 64`.
3. **Fallas al compilar PyBullet en Windows:**
   * Asegúrate de ejecutar la simulación desde dentro del entorno virtual `.venv` utilizando Python 3.11.
