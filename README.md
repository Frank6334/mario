import pygame
import random

# Inicializar Pygame
pygame.init()

# Dimensiones de la pantalla
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Mario Bros - Juego")

# Colores
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
YELLOW = (255, 215, 0)
BROWN = (139, 69, 19)
WHITE = (255, 255, 255)
FLAG_COLOR = (255, 0, 255)

# Clase Mario
class Mario:
    def __init__(self):
        self.x = 100
        self.y = HEIGHT - 100
        self.width = 40
        self.height = 50
        self.vel_x = 5
        self.vel_y = 0
        self.gravity = 0.6
        self.jump_power = -12  # Más negativo = más alto
        self.on_ground = False

    def move(self, keys):
        if keys[pygame.K_LEFT]:
            self.x -= self.vel_x
        if keys[pygame.K_RIGHT]:
            self.x += self.vel_x
        # SALTO con Enter o Espacio
        if (keys[pygame.K_SPACE] or keys[pygame.K_RETURN]) and self.on_ground:
            self.vel_y = self.jump_power
            self.on_ground = False

    def apply_gravity(self, structures):
        self.vel_y += self.gravity
        self.y += self.vel_y

        # Detectar colisión con el suelo
        if self.y >= HEIGHT - 50:
            self.y = HEIGHT - 50
            self.vel_y = 0
            self.on_ground = True

        # Detectar colisión con estructuras (para poder subir)
        for struct in structures:
            if (self.x + self.width > struct.x and self.x < struct.x + struct.width and
                self.y + self.height <= struct.y and self.y + self.height + self.vel_y >= struct.y):
                self.y = struct.y - self.height
                self.vel_y = 0
                self.on_ground = True

    def draw(self):
        pygame.draw.rect(screen, RED, (self.x + 10, self.y, 20, 10))  # Gorra
        pygame.draw.rect(screen, BLUE, (self.x, self.y + 10, 40, 30))  # Cuerpo
        pygame.draw.rect(screen, BROWN, (self.x + 5, self.y + 40, 10, 10))  # Pierna izquierda
        pygame.draw.rect(screen, BROWN, (self.x + 25, self.y + 40, 10, 10))  # Pierna derecha

# Clase Estructura
class Structure:
    def __init__(self, x, y, width, height):
        self.x = x
        self.y = y
        self.width = width
        self.height = height

    def draw(self):
        pygame.draw.rect(screen, BROWN, (self.x, self.y, self.width, self.height))

# Clase Enemigo (Goomba)
class Goomba:
    def __init__(self, x, y, speed):
        self.x = x
        self.y = y
        self.speed = speed
        self.width = 20
        self.height = 20

    def move(self):
        self.x += self.speed
        if self.x <= 0 or self.x + self.width >= WIDTH:
            self.speed *= -1

    def draw(self):
        pygame.draw.rect(screen, BROWN, (self.x, self.y, self.width, self.height))

# Clase Meta (Bandera)
class Goal:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.width = 20
        self.height = 100

    def draw(self):
        pygame.draw.rect(screen, FLAG_COLOR, (self.x, self.y, self.width, self.height))

# Función para dibujar el botón de reinicio
def draw_restart_button():
    font = pygame.font.SysFont("Arial", 30)
    text = font.render("Volver a intentar", True, WHITE)
    button_rect = pygame.Rect(WIDTH // 2 - 150, HEIGHT // 2 - 30, 300, 60)
    pygame.draw.rect(screen, RED, button_rect)
    screen.blit(text, (WIDTH // 2 - 140, HEIGHT // 2 - 20))
    return button_rect

# Función para generar estructuras aleatorias
def generate_random_structures(num_structures):
    structures = []
    for _ in range(num_structures):
        x = random.randint(0, WIDTH - 150)
        y = random.randint(100, HEIGHT - 100)
        width = random.randint(100, 200)
        height = 20
        structures.append(Structure(x, y, width, height))
    return structures

# Configurar elementos del juego
goombas = [Goomba(random.randint(300, 700), HEIGHT - 50, random.choice([-3, 3])) for _ in range(3)]
goal = Goal(750, HEIGHT - 150)
mario = Mario()
structures = generate_random_structures(5)

# Bucle del juego
running = True
clock = pygame.time.Clock()
game_over = False
won = False

while running:
    clock.tick(60)
    screen.fill(BLACK)

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        if event.type == pygame.MOUSEBUTTONDOWN and game_over:
            mouse_x, mouse_y = event.pos
            if restart_button.collidepoint(mouse_x, mouse_y):
                # Reiniciar el juego
                mario = Mario()
                goombas = [Goomba(random.randint(300, 700), HEIGHT - 50, random.choice([-3, 3])) for _ in range(3)]
                structures = generate_random_structures(5)
                game_over = False
                won = False

    if not game_over:
        keys = pygame.key.get_pressed()
        mario.move(keys)
        mario.apply_gravity(structures)

        for goomba in goombas:
            goomba.move()

        # Dibujar estructuras
        for struct in structures:
            struct.draw()

        goal.draw()
        mario.draw()
        for goomba in goombas:
            goomba.draw()

        # Verificar si Mario alcanzó la meta
        if mario.x + mario.width > goal.x and mario.y < goal.y + goal.height:
            won = True
            game_over = True

        # Verificar colisión con Goombas
        for goomba in goombas:
            if (mario.x + mario.width > goomba.x and mario.x < goomba.x + goomba.width and
                mario.y + mario.height > goomba.y and mario.y < goomba.y + goomba.height):
                game_over = True

    if game_over:
        if won:
            font = pygame.font.SysFont("Arial", 50)
            screen.blit(font.render("¡Felicidades, ganaste!", True, WHITE), (WIDTH // 2 - 170, HEIGHT // 2 - 100))
        else:
            restart_button = draw_restart_button()

    pygame.display.flip()

pygame.quit()
