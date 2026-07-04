import pygame
import sys
import copy

# ==============================================================================
# CONFIGURACIONES GENERALES Y CONSTANTES
# ==============================================================================
# Dimensiones de la matriz del mapa
ROWS, COLS = 15, 19
TILE_SIZE = 40

# Dimensiones de la pantalla
SCREEN_WIDTH = COLS * TILE_SIZE
SCREEN_HEIGHT = (ROWS * TILE_SIZE) + 50  # Espacio extra abajo para el score

# Colores (RGB)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)

# Direcciones (Ejes: x, y)
DIR_UP = (0, -1)
DIR_DOWN = (0, 1)
DIR_LEFT = (-1, 0)
DIR_RIGHT = (1, 0)

# El mapa inicial: 
# 1 = Pared, 0 = Píldora (punto), 2 = Vacío (camino libre)
INITIAL_MAP = [
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],
    [1,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,1],
    [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
    [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
    [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
    [1,0,1,1,0,1,0,1,1,1,1,1,0,1,0,1,1,0,1],
    [1,0,0,0,0,1,0,0,0,1,0,0,0,1,0,0,0,0,1],
    [1,1,1,1,0,1,1,1,2,1,2,1,1,1,0,1,1,1,1],
    [1,0,0,0,0,1,0,0,0,2,0,0,0,1,0,0,0,0,1],
    [1,0,1,1,0,1,0,1,1,1,1,1,0,1,0,1,1,0,1],
    [1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1],
    [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
    [1,0,1,1,0,1,1,1,0,1,0,1,1,1,0,1,1,0,1],
    [1,0,0,0,0,0,0,0,0,1,0,0,0,0,0,0,0,0,1],
    [1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1]
]

# ==============================================================================
# CLASES PRINCIPALES
# ==============================================================================

class Pacman:
    def __init__(self, x, y):
        self.grid_x = x
        self.grid_y = y
        # Posición en pixeles basada en el centro de la celda
        self.pix_x = x * TILE_SIZE + TILE_SIZE // 2
        self.pix_y = y * TILE_SIZE + TILE_SIZE // 2
        self.direction = DIR_RIGHT
        self.next_direction = DIR_RIGHT
        self.speed = 2
        self.mouth_angle = 0
        self.mouth_closing = False

    def handle_input(self):
        keys = pygame.key.get_pressed()
        if keys[pygame.K_UP] or keys[pygame.K_w]:
            self.next_direction = DIR_UP
        elif keys[pygame.K_DOWN] or keys[pygame.K_s]:
            self.next_direction = DIR_DOWN
        elif keys[pygame.K_LEFT] or keys[pygame.K_a]:
            self.next_direction = DIR_LEFT
        elif keys[pygame.K_RIGHT] or keys[pygame.K_d]:
            self.next_direction = DIR_RIGHT

    def can_move(self, direction, game_map):
        # Calcula la siguiente celda en la matriz
        next_x = self.grid_x + direction[0]
        next_y = self.grid_y + direction[1]
        
        # Evita salir de los límites de la matriz
        if 0 <= next_x < COLS and 0 <= next_y < ROWS:
            return game_map[next_y][next_x] != 1
        return False

    def update(self, game_map):
        # Control de la animación de la boca
        if self.mouth_closing:
            self.mouth_angle -= 2
            if self.mouth_angle <= 0:
                self.mouth_closing = False
        else:
            self.mouth_angle += 2
            if self.mouth_angle >= 30:
                self.mouth_closing = True

        # Si está perfectamente alineado con la cuadrícula, evalúa cambiar de dirección
        if (self.pix_x - TILE_SIZE // 2) % TILE_SIZE == 0 and (self.pix_y - TILE_SIZE // 2) % TILE_SIZE == 0:
            self.grid_x = (self.pix_x - TILE_SIZE // 2) // TILE_SIZE
            self.grid_y = (self.pix_y - TILE_SIZE // 2) // TILE_SIZE

            if self.can_move(self.next_direction, game_map):
                self.direction = self.next_direction

            if not self.can_move(self.direction, game_map):
                return  # Se detiene si hay pared al frente

        # Movimiento continuo en pixeles
        self.pix_x += self.direction[0] * self.speed
        self.pix_y += self.direction[1] * self.speed

    def draw(self, screen):
        # Dibuja a Pac-Man usando un círculo simple
        pygame.draw.circle(screen, YELLOW, (self.pix_x, self.pix_y), TILE_SIZE // 2 - 2)


class Ghost:
    def __init__(self, x, y):
        self.grid_x = x
        self.grid_y = y
        self.pix_x = x * TILE_SIZE + TILE_SIZE // 2
        self.pix_y = y * TILE_SIZE + TILE_SIZE // 2
        self.direction = DIR_UP
        self.speed = 2

    def update(self, game_map, target_x, target_y):
        # Cuando está alineado a la cuadrícula, decide su próximo movimiento
        if (self.pix_x - TILE_SIZE // 2) % TILE_SIZE == 0 and (self.pix_y - TILE_SIZE // 2) % TILE_SIZE == 0:
            self.grid_x = (self.pix_x - TILE_SIZE // 2) // TILE_SIZE
            self.grid_y = (self.pix_y - TILE_SIZE // 2) // TILE_SIZE

            # IA Básica: Elige la dirección válida que reduzca la distancia física a Pac-Man
            possible_directions = [DIR_UP, DIR_DOWN, DIR_LEFT, DIR_RIGHT]
            best_dir = self.direction
            min_distance = float('inf')

            for d in possible_directions:
                # Evita que regrese inmediatamente sobre sus pasos de forma directa
                if d[0] == -self.direction[0] and d[1] == -self.direction[1]:
                    continue
                
                next_x = self.grid_x + d[0]
                next_y = self.grid_y + d[1]

                if 0 <= next_x < COLS and 0 <= next_y < ROWS and game_map[next_y][next_x] != 1:
                    # Distancia Manhattan o geométrica simple al objetivo
                    dist = abs(next_x - target_x) + abs(next_y - target_y)
                    if dist < min_distance:
                        min_distance = dist
                        best_dir = d

            self.direction = best_dir

            # Si se bloquea, busca cualquier salida válida
            next_x = self.grid_x + self.direction[0]
            next_y = self.grid_y + self.direction[1]
            if game_map[next_y][next_x] == 1:
                for d in [DIR_UP, DIR_DOWN, DIR_LEFT, DIR_RIGHT]:
                    if game_map[self.grid_y + d[1]][self.grid_x + d[0]] != 1:
                        self.direction = d
                        break

        self.pix_x += self.direction[0] * self.speed
        self.pix_y += self.direction[1] * self.speed

    def draw(self, screen):
        # Cuerpo principal del fantasma (Círculo + Rectángulo abajo)
        radius = TILE_SIZE // 2 - 2
        pygame.draw.circle(screen, RED, (self.pix_x, self.pix_y), radius)
        pygame.draw.rect(screen, RED, (self.pix_x - radius, self.pix_y, radius * 2, radius))
        
        # Ojos básicos
        pygame.draw.circle(screen, WHITE, (self.pix_x - 5, self.pix_y - 4), 4)
        pygame.draw.circle(screen, WHITE, (self.pix_x + 5, self.pix_y - 4), 4)


# ==============================================================================
# MOTOR PRINCIPAL DEL JUEGO (GAME LOOP)
# ==============================================================================

class Game:
    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
        pygame.display.set_caption("Pac-Man Open Source Clone")
        self.clock = pygame.time.Clock()
        self.font = pygame.font.SysFont("Arial", 24)
        
        self.game_map = copy.deepcopy(INITIAL_MAP)
        self.pacman = Pacman(9, 10)
        self.ghost = Ghost(9, 7)
        self.score = 0
        self.game_over = False
        self.won = False

    def check_collisions(self):
        # Colisión con píldoras
        p_grid_x = (self.pacman.pix_x - TILE_SIZE // 2) // TILE_SIZE
        p_grid_y = (self.pacman.pix_y - TILE_SIZE // 2) // TILE_SIZE

        if 0 <= p_grid_x < COLS and 0 <= p_grid_y < ROWS:
            if self.game_map[p_grid_y][p_grid_x] == 0:
                self.game_map[p_grid_y][p_grid_x] = 2  # Cambia a vacío
                self.score += 10

        # Verificar si quedan píldoras en el mapa
        pills_left = any(0 in row for row in self.game_map)
        if not pills_left:
            self.won = True
            self.game_over = True

        # Colisión con el fantasma (basada en la proximidad en pixeles)
        distance = ((self.pacman.pix_x - self.ghost.pix_x) ** 2 + (self.pacman.pix_y - self.ghost.pix_y) ** 2) ** 0.5
        if distance < TILE_SIZE // 1.5:
            self.game_over = True

    def draw_map(self):
        for r in range(ROWS):
            for c in range(COLS):
                tile = self.game_map[r][c]
                x = c * TILE_SIZE
                y = r * TILE_SIZE
                
                if tile == 1:
                    # Dibuja bloques de pared azules
                    pygame.draw.rect(self.screen, BLUE, (x + 2, y + 2, TILE_SIZE - 4, TILE_SIZE - 4), 2)
                elif tile == 0:
                    # Dibuja los puntos pequeños
                    pygame.draw.circle(self.screen, WHITE, (x + TILE_SIZE // 2, y + TILE_SIZE // 2), 4)

    def draw_ui(self):
        # Renderiza el puntaje en la parte inferior
        score_text = self.font.render(f"SCORE: {self.score}", True, WHITE)
        self.screen.blit(score_text, (15, SCREEN_HEIGHT - 40))

        if self.game_over:
            msg = "¡GANASTE!" if self.won else "GAME OVER"
            color = YELLOW if self.won else RED
            over_text = self.font.render(msg, True, color)
            self.screen.blit(over_text, (SCREEN_WIDTH // 2 - 60, SCREEN_HEIGHT - 40))

    def run(self):
        while True:
            # Manejo de eventos del sistema
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    pygame.quit()
                    sys.exit()

            # Lógica del juego si no se ha terminado
            if not self.game_over:
                self.pacman.handle_input()
                self.pacman.update(self.game_map)
                self.ghost.update(self.game_map, self.pacman.grid_x, self.pacman.grid_y)
                self.check_collisions()

            # Renderizado
            self.screen.fill(BLACK)
            self.draw_map()
            self.pacman.draw(self.screen)
            self.ghost.draw(self.screen)
            self.draw_ui()

            pygame.display.flip()
            self.clock.tick(60)  # Forzar a 60 FPS

if __name__ == "__main__":
    game = Game()
    game.run()
