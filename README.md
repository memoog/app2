import pygame
import sys
import math
from pygame.locals import *

# Configuración inicial
pygame.init()
pygame.font.init()
font = pygame.font.SysFont("Arial", 20)
clock = pygame.time.Clock()

# Dimensiones iniciales de la ventana
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT), RESIZABLE)
pygame.display.set_caption("Simulación de Vitrectomía 3D – Versión Python")

# Colores
BLACK = (0, 0, 0)
DARK_RED = (50, 0, 0)
RED = (255, 0, 0)
WHITE = (255, 255, 255)
GRAY = (100, 100, 100)
LIGHT_BLUE = (50, 150, 255)

# Parámetros de la "retina"
retina_center = (WIDTH // 2, HEIGHT // 2)
retina_radius = min(WIDTH, HEIGHT) // 3

# Instrumentos: se dibujan como rectángulos
vitrectomo_size = (int(retina_radius * 0.1), int(retina_radius * 0.5))
endo_size = (20, 150)

# Posiciones iniciales (normalizadas de 0 a 1)
joystick_vit_pos = [0.5, 0.5]  # centro de la retina
joystick_endo_pos = [0.5, 0.5]

# Variables para alertas
alert_message = ""
alert_timer = 0

def draw_retina(surface, center, radius):
    """Dibuja una retina con efecto degradado (simulado mediante varios círculos)."""
    steps = 20
    for i in range(steps, 0, -1):
        color_val = 50 + int(205 * (i / steps))
        color = (color_val, 0, 0)
        pygame.draw.circle(surface, color, center, int(radius * i / steps))

def draw_joystick(surface, pos, base_radius=40):
    """Dibuja un joystick: un círculo base y un “handle” que se mueve."""
    # Base
    pygame.draw.circle(surface, GRAY, pos, base_radius, 3)
    # Handle (circulo central)
    handle_radius = base_radius // 2
    pygame.draw.circle(surface, LIGHT_BLUE, pos, handle_radius)

def get_screen_positions():
    """Calcula las posiciones actualizadas de la retina e instrumentos según el tamaño actual de la ventana."""
    current_width, current_height = screen.get_size()
    retina_center = (current_width // 2, current_height // 2)
    retina_radius = min(current_width, current_height) // 3
    return retina_center, retina_radius

def draw_instruments(surface, retina_center, retina_radius):
    """Dibuja los instrumentos dentro de la retina según la posición del joystick."""
    # Vitrectomo: se mueve según joystick_vit_pos
    vit_x = retina_center[0] + int((joystick_vit_pos[0] - 0.5) * retina_radius * 2)
    vit_y = retina_center[1] + int((joystick_vit_pos[1] - 0.5) * retina_radius * 2)
    vit_rect = pygame.Rect(0, 0, vitrectomo_size[0], vitrectomo_size[1])
    vit_rect.center = (vit_x, vit_y)
    pygame.draw.rect(surface, WHITE, vit_rect)
    
    # Endoiluminador: posición fija en la parte inferior izquierda (pero se simula el "handle" con joystick)
    endo_x = int(0.1 * retina_radius)
    endo_y = retina_center[1] + retina_radius - 20
    # Para simular control, desplazamos el “punto de luz” con joystick_endo_pos (muy sutil)
    offset_x = int((joystick_endo_pos[0] - 0.5) * 20)
    offset_y = int((joystick_endo_pos[1] - 0.5) * 20)
    endo_rect = pygame.Rect(0, 0, endo_size[0], endo_size[1])
    endo_rect.center = (endo_x + offset_x, endo_y + offset_y)
    pygame.draw.rect(surface, GRAY, endo_rect)

def check_alerts(retina_center, retina_radius, vit_pos):
    """Verifica si el vitrectomo ha chocado con el borde de la retina y retorna un mensaje de alerta."""
    global alert_message, alert_timer
    # Calcula distancia del vitrectomo (posicionado según joystick_vit_pos) al centro de la retina
    vit_x = retina_center[0] + int((joystick_vit_pos[0] - 0.5) * retina_radius * 2)
    vit_y = retina_center[1] + int((joystick_vit_pos[1] - 0.5) * retina_radius * 2)
    dx = vit_x - retina_center[0]
    dy = vit_y - retina_center[1]
    distance = math.hypot(dx, dy)
    # Si el instrumento se acerca al borde (más del 85% del radio), se considera colisión
    if distance > retina_radius * 0.85:
        alert_message = "¡El Vitrectomo ha chocado con la pared del ojo!"
        alert_timer = pygame.time.get_ticks()
    else:
        # También se puede establecer otras condiciones de alerta (por ejemplo, presión, profundidad, etc.)
        alert_message = ""

def draw_alert(surface):
    """Dibuja el mensaje de alerta en la parte superior de la pantalla si existe y si no ha expirado (2 segundos)."""
    if alert_message:
        current_time = pygame.time.get_ticks()
        if current_time - alert_timer < 2000:
            text_surface = font.render(alert_message, True, WHITE)
            surface.blit(text_surface, (20, 20))

def handle_joystick_input(mouse_pos, joystick_center, joystick_radius):
    """Devuelve la posición normalizada (0 a 1) según la posición del mouse en un área circular."""
    dx = mouse_pos[0] - joystick_center[0]
    dy = mouse_pos[1] - joystick_center[1]
    distance = math.hypot(dx, dy)
    # Limitar al radio del joystick
    if distance > joystick_radius:
        scale = joystick_radius / distance
        dx *= scale
        dy *= scale
    norm_x = (dx / joystick_radius + 1) / 2
    norm_y = (dy / joystick_radius + 1) / 2
    return norm_x, norm_y

# Estados para saber si estamos manipulando cada joystick
moving_vit = False
moving_endo = False

# Áreas para detectar clicks sobre los joysticks (se ubican en la parte inferior)
def get_joystick_areas(retina_center, retina_radius):
    # En este ejemplo, definimos dos áreas circulares para los joysticks en la parte inferior de la pantalla
    current_width, current_height = screen.get_size()
    # Vitrectomo en la esquina inferior izquierda
    vit_area_center = (int(0.15 * current_width), current_height - 100)
    endo_area_center = (int(0.85 * current_width), current_height - 100)
    joystick_radius = 50
    return vit_area_center, endo_area_center, joystick_radius

# Bucle principal
running = True
while running:
    screen.fill(BLACK)
    current_width, current_height = screen.get_size()
    retina_center, retina_radius = get_screen_positions()
    
    # Dibuja la retina (simula degradado)
    draw_retina(screen, retina_center, retina_radius)
    
    # Dibuja los instrumentos en la retina
    draw_instruments(screen, retina_center, retina_radius)
    
    # Dibuja el mini mapa (una representación simple en la parte superior)
    pygame.draw.rect(screen, GRAY, (20, 20, 150, 100), 2)
    mini_text = font.render("Mini Mapa", True, WHITE)
    screen.blit(mini_text, (25, 25))
    
    # Obtiene las áreas para los joysticks
    vit_area_center, endo_area_center, joy_radius = get_joystick_areas(retina_center, retina_radius)
    # Dibuja las bases de los joysticks
    draw_joystick(screen, vit_area_center, joy_radius)
    draw_joystick(screen, endo_area_center, joy_radius)
    
    # Dibuja alertas (si hay)
    check_alerts(retina_center, retina_radius, joystick_vit_pos)
    draw_alert(screen)
    
    for event in pygame.event.get():
        if event.type == QUIT:
            running = False
            
        elif event.type == VIDEORESIZE:
            screen = pygame.display.set_mode((event.w, event.h), RESIZABLE)
            
        elif event.type == MOUSEBUTTONDOWN:
            mouse_pos = event.pos
            # Comprueba si el click está en el área del joystick del vitrectomo
            if math.hypot(mouse_pos[0] - vit_area_center[0], mouse_pos[1] - vit_area_center[1]) <= joy_radius:
                moving_vit = True
            # Comprueba si el click está en el área del joystick del endoiluminador
            elif math.hypot(mouse_pos[0] - endo_area_center[0], mouse_pos[1] - endo_area_center[1]) <= joy_radius:
                moving_endo = True
                
        elif event.type == MOUSEBUTTONUP:
            moving_vit = False
            moving_endo = False
            
        elif event.type == MOUSEMOTION:
            mouse_pos = event.pos
            if moving_vit:
                joystick_vit_pos = list(handle_joystick_input(mouse_pos, vit_area_center, joy_radius))
            if moving_endo:
                joystick_endo_pos = list(handle_joystick_input(mouse_pos, endo_area_center, joy_radius))
    
    pygame.display.flip()
    clock.tick(60)

pygame.quit()
sys.exit()
