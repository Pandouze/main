# main
encore
import pygame
import random
import json
import os
import time

# ======================
# CONFIG
# ======================
WIDTH, HEIGHT = 800, 600
FPS = 60
SAVE_FILE = "memoire.json"

# ======================
# INIT
# ======================
pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Encore Mort, Chérie.")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 18)
big_font = pygame.font.SysFont("arial", 28)

# ======================
# SAVE / MEMOIRE
# ======================
def load_memory():
    if os.path.exists(SAVE_FILE):
        with open(SAVE_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    return {
        "deaths": 0,
        "runs": 0,
        "insults_unlocked": 0
    }

def save_memory(mem):
    with open(SAVE_FILE, "w", encoding="utf-8") as f:
        json.dump(mem, f, indent=2, ensure_ascii=False)

memory = load_memory()

# ======================
# NARRATEUR
# ======================
INSULTS = [
    "On va faire comme si c'était stratégique.",
    "Tu savais que courir droit dans l'ennemi n'est pas un build ?",
    "J'admire ta constance. Pas ton talent.",
    "Encore ce mur. Il te connaît maintenant.",
    "C'était audacieux. Inutile, mais audacieux.",
    "Tu progresses. Lentement. Très lentement.",
    "Ce bouton-là, tu l'as appuyé par accident, avoue.",
    "La panique n'est pas une mécanique. Enfin… chez toi si.",
]

def narrator_comment(event):
    base = random.choice(INSULTS[:max(1, memory["insults_unlocked"] + 1)])
    if event == "death":
        return f"{base} (Mort n°{memory['deaths']})"
    if event == "start":
        return f"Nouvelle run. Essaie de ne pas mourir en 10 secondes."
    return base

# ======================
# PLAYER
# ======================
class Player:
    def __init__(self):
        self.rect = pygame.Rect(WIDTH//2, HEIGHT//2, 30, 30)
        self.speed = 4
        self.hp = 3

    def move(self, keys):
        dx = dy = 0
        if keys[pygame.K_z]: dy -= self.speed
        if keys[pygame.K_s]: dy += self.speed
        if keys[pygame.K_q]: dx -= self.speed
        if keys[pygame.K_d]: dx += self.speed
        self.rect.move_ip(dx, dy)
        self.rect.clamp_ip(screen.get_rect())

    def draw(self):
        pygame.draw.rect(screen, (255, 105, 180), self.rect)

# ======================
# ENEMY
# ======================
class Enemy:
    def __init__(self):
        x = random.randint(0, WIDTH)
        y = random.randint(0, HEIGHT)
        self.rect = pygame.Rect(x, y, 25, 25)
        self.speed = random.choice([1, 2])

    def update(self, player):
        if self.rect.x < player.rect.x:
            self.rect.x += self.speed
        if self.rect.x > player.rect.x:
            self.rect.x -= self.speed
        if self.rect.y < player.rect.y:
            self.rect.y += self.speed
        if self.rect.y > player.rect.y:
            self.rect.y -= self.speed

    def draw(self):
        pygame.draw.rect(screen, (180, 180, 180), self.rect)

# ======================
# GAME STATE
# ======================
def game_loop():
    global memory

    player = Player()
    enemies = [Enemy() for _ in range(3 + memory["runs"])]
    comment = narrator_comment("start")
    comment_timer = time.time()

    running = True
    while running:
        clock.tick(FPS)
        screen.fill((20, 20, 20))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                save_memory(memory)
                pygame.quit()
                exit()

        keys = pygame.key.get_pressed()
        player.move(keys)

        for enemy in enemies:
            enemy.update(player)
            if enemy.rect.colliderect(player.rect):
                memory["deaths"] += 1
                memory["insults_unlocked"] = min(len(INSULTS)-1, memory["deaths"]//2)
                save_memory(memory)
                return False  # mort

        player.draw()
        for enemy in enemies:
            enemy.draw()

        # UI
        hp_text = font.render(f"HP: {player.hp}", True, (255, 255, 255))
        screen.blit(hp_text, (10, 10))

        if time.time() - comment_timer < 3:
            txt = big_font.render(comment, True, (255, 255, 255))
            screen.blit(txt, (WIDTH//2 - txt.get_width()//2, HEIGHT - 50))

        pygame.display.flip()

    return True

# ======================
# MAIN
# ======================
while True:
    memory["runs"] += 1
    alive = game_loop()

    screen.fill((0, 0, 0))
    msg = narrator_comment("death")
    txt = big_font.render(msg, True, (255, 255, 255))
    screen.blit(txt, (WIDTH//2 - txt.get_width()//2, HEIGHT//2))
    pygame.display.flip()

    pygame.time.wait(2500)
