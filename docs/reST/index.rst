import pygame
import sys

# Initialize pygame
pygame.init()

# Constants
WIDTH, HEIGHT = 800, 600
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
FONT = pygame.font.Font(None, 36)

# Game states
MENU = "menu"
CUTSCENE = "cutscene"
GAME = "game"
INVENTORY = "inventory"

# Load images
player_img = pygame.Surface((40, 60))
player_img.fill((0, 255, 0))

enemy_img = pygame.Surface((40, 60))
enemy_img.fill((255, 0, 0))

background_img = pygame.Surface((WIDTH, HEIGHT))
background_img.fill((150, 150, 150))

# Initialize screen
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Piggy-Inspired Game")

# Inventory
inventory = []

# Player class
class Player(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = player_img
        self.rect = self.image.get_rect(topleft=(x, y))
        self.speed = 4

    def update(self, keys):
        if keys[pygame.K_LEFT]:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT]:
            self.rect.x += self.speed
        if keys[pygame.K_UP]:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN]:
            self.rect.y += self.speed

# Enemy (Piggy) class
class Enemy(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = enemy_img
        self.rect = self.image.get_rect(topleft=(x, y))

    def update(self, player):
        if player.rect.x > self.rect.x:
            self.rect.x += 2
        elif player.rect.x < self.rect.x:
            self.rect.x -= 2
        if player.rect.y > self.rect.y:
            self.rect.y += 2
        elif player.rect.y < self.rect.y:
            self.rect.y -= 2

# Key item
class Key(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((20, 20))
        self.image.fill((255, 255, 0))
        self.rect = self.image.get_rect(topleft=(x, y))

# Scene Manager
class GameManager:
    def __init__(self):
        self.state = MENU
        self.player = Player(100, 100)
        self.enemy = Enemy(500, 300)
        self.key = Key(400, 400)
        self.all_sprites = pygame.sprite.Group(self.player, self.enemy, self.key)

    def start_cutscene(self):
        self.state = CUTSCENE

    def start_game(self):
        self.state = GAME

    def open_inventory(self):
        self.state = INVENTORY

    def handle_inventory(self, item):
        if item == "key":
            inventory.append("Key")
            self.key.kill()

    def update(self, keys):
        if self.state == GAME:
            self.all_sprites.update(keys)
            if pygame.sprite.collide_rect(self.player, self.key):
                self.handle_inventory("key")

    def draw(self):
        screen.fill(WHITE)

        if self.state == MENU:
            menu_text = FONT.render("Press ENTER to Start", True, BLACK)
            screen.blit(menu_text, (WIDTH // 2 - 100, HEIGHT // 2))

        elif self.state == CUTSCENE:
            cutscene_text = FONT.render("A dark night... something is lurking.", True, BLACK)
            screen.blit(cutscene_text, (WIDTH // 2 - 150, HEIGHT // 2))

        elif self.state == GAME:
            screen.blit(background_img, (0, 0))
            self.all_sprites.draw(screen)

        elif self.state == INVENTORY:
            inventory_text = FONT.render(f"Inventory: {', '.join(inventory)}", True, BLACK)
            screen.blit(inventory_text, (WIDTH // 2 - 100, HEIGHT // 2))

# Main function
def main():
    clock = pygame.time.Clock()
    game = GameManager()
    
    running = True
    while running:
        keys = pygame.key.get_pressed()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False

            if event.type == pygame.KEYDOWN:
                if game.state == MENU and event.key == pygame.K_RETURN:
                    game.start_cutscene()
                elif game.state == CUTSCENE and event.key == pygame.K_RETURN:
                    game.start_game()
                elif event.key == pygame.K_i:
                    game.open_inventory()

        game.update(keys)
        game.draw()

        pygame.display.flip()
        clock.tick(60)

    pygame.quit()
    sys.exit()

if __name__ == "__main__":
    main()
