# 🕹️ Pac-Man Open Source Clone

¡Bienvenido a este clon clásico de **Pac-Man** desarrollado en Python utilizando la librería **Pygame**! Este proyecto fue diseñado con un enfoque educativo y estructurado para demostrar las bases del desarrollo de videojuegos en 2D, el manejo de matrices para el diseño de mapas y una Inteligencia Artificial básica para el comportamiento de los enemigos.

---

## ✨ Características del Proyecto

* **Movimiento Continuo y Fluido**: Control preciso basado en una cuadrícula de píxeles mediante las flechas del teclado o las teclas `WASD`.
* **IA del Fantasma (Persecución)**: El enemigo evalúa constantemente su entorno para tomar la ruta óptima en la cuadrícula que reduzca la distancia física hacia Pac-Man.
* **Sistema de Puntuación**: Mecánica de colisiones precisa para recolectar píldoras y actualización del *Score* en tiempo real.
* **Código Limpio y Modular**: Separación de responsabilidades mediante programación orientada a objetos (`Pacman`, `Ghost`, `Game`).

---

## 🛠️ Tecnologías Utilizadas

* **Python 3.10+**
* **Pygame**: Biblioteca principal para el renderizado de gráficos 2D y manejo de eventos.
* **Copy**: Módulo estándar de Python utilizado para gestionar de forma segura el reinicio de las matrices del mapa.

---

## 📁 Estructura del Repositorio

El proyecto cuenta con una arquitectura simple y directa:

```text
├── main.py            # Código fuente principal con toda la lógica del juego.
├── requirements.txt   # Archivo de dependencias del proyecto.
└── README.md          # Documentación del repositorio.
