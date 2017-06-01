# Raiden
this is raiden
import pygame, sys, time, os

# Colors
SKY_BLUE = (135, 206, 250)
GREEN = (34, 139, 34)
WHITE = (255, 255, 255)
BROWN = (139, 69, 19)
GREY = (190, 190, 190)
RED = (255, 0, 0)
LIGHT_BLUE = (0, 191, 255)
BLUE = (0, 0, 255)
LIGHT_GREY = (211, 211, 211)

#os.environ['SDL_VIDEO_CENTERED'] = '1'

WIN_W = 16*32
WIN_H = 700

SHIP_WIDTH = WIN_W / 15
SHIP_HEIGHT = WIN_H / 15

TIMER = 0

# Constants
class Platform(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.Surface((32, 32))
        self.image.convert()
        self.image.fill(LIGHT_GREY)
        self.rect = pygame.Rect(x, y, 32, 32)


class Ship(pygame.sprite.Sprite):
    def __init__(self, container):
        pygame.sprite.Sprite.__init__(self)
        self.side_speed = 6
        self.top_speed = 5
        self.image = pygame.Surface((SHIP_WIDTH, SHIP_HEIGHT)).convert()
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.rect.centerx = container.centerx
        self.rect.bottom = container.bottom
        self.rect.y = container.height - self.rect.height * 3
        self.container = container
        self.bob = None
        self.bullet = None

    def update(self, camera_entity, bullet_group):
        key = pygame.key.get_pressed()
        self.rect.clamp_ip(self.container)
        if key[pygame.K_w]:
            self.rect.y -= self.top_speed
        if key[pygame.K_s]:
            self.rect.y += self.top_speed
        if key[pygame.K_a]:
            self.rect.x -= self.side_speed
        if key[pygame.K_d]:
            self.rect.x += self.side_speed

        if key[pygame.K_SPACE]:
            global TIMER
            TIMER += 1
            if TIMER % 10 == 0:
                bullet = Bullet(self)
                bullet_group.add(bullet)
                bullet_group.update(camera_entity)


        if camera_entity.rect.y < 690 and camera_entity.rect.y > WIN_H / 2:
            self.bob = True
        else:
            self.bob = False

        if self.bob:
                self.rect.y -= 1


class Camera(object):
    def __init__(self, total_width, total_height):
        self.state = pygame.Rect(0, 0, total_width, total_height)
        self.image = pygame.Surface((1,1))
        self.rect = self.image.get_rect()
        self.y = self.rect.y

    def apply(self, target):
        return target.rect.move(self.state.topleft)

    def update(self, camera_entity_rect, ship_rect):
        x = self.ship_camera(ship_rect)
        y = self.level_camera(camera_entity_rect)

        self.state = pygame.Rect(x, y, self.state.width, self.state.height)

    def level_camera(self, camera_entity_rect):
        # Y calculation is done here

        y = -camera_entity_rect.y + WIN_H / 2

        # Stop scrolling at top
        if y > 0:
            y = 0
        # Stop scrolling at the bottom
        elif y < -(self.state.height - WIN_H):
            y = -(self.state.height - WIN_H)

        return y

    def ship_camera(self, ship_rect):
        # X calculation is done here.

        x = -ship_rect.x + WIN_W / 2

        # Stop scrolling at left edge
        if x > 0:
            x = 0
        # Stop scrolling at the right edge
        elif x < -(self.state.width - WIN_W):
            x = -(self.state.width - WIN_W)
        return x


class CameraEntity(pygame.sprite.Sprite):
    def __init__(self, total_rect):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.Surface((20, 20))
        self.rect = self.image.get_rect()
        self.rect.centerx = total_rect.centerx
        self.rect.bottom = total_rect.bottom
        self.speed = [0, -1]
        self.move = True
        self.image.fill(BLUE)

    def update(self, camera_entity):
        if self.rect.y == WIN_H / 2:
            self.move = False

        if self.move:
            self.rect = self.rect.move(self.speed)


class Bullet(pygame.sprite.Sprite):
    def __init__(self, ship):
        pygame.sprite.Sprite.__init__(self)
        self.speed = -15
        self.image = pygame.Surface((5, 5))
        self.image.fill(GREEN)
        self.rect = self.image.get_rect()
        self.set_pos(ship)

    def set_pos(self, ship):
        self.rect.x = ship.rect.x + (SHIP_WIDTH / 2)
        self.rect.y = ship.rect.y

    def update(self, camera_entity):
        if self.rect.y > 0:
            self.rect.y += self.speed
        if self.rect.y <= 0:
            self.kill()


def main():
    pygame.init()

    # Create Game Variables
    fps = 60
    clock = pygame.time.Clock()
    play = True
    pygame.display.set_caption('Platformer')
    screen = pygame.display.set_mode((WIN_W, WIN_H), pygame.SRCALPHA)

    # Load Level
    level = [
        "PPPPPPPPPPPPPPPPPPPPPP",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                PPPPP",
        "P                    P",
        "PPPPPPPP             P",
        "P                    P",
        "P                    P",
        "P                    P",
        "P                PPPPP",
        "P                    P",
        "P                    P",
        "PPPP                 P",
        "P                    P",
        "P                 PPPP",
        "PPPPP                P",
        "P                    P",
        "P                    P",
        "P              PPPPPPP",
        "P                    P",
        "PPPPPP               P",
        "P                    P",
        "PPPPPPPPPPPPPPPPPPPPPP", ]

    # Create Groups
    platform_group = pygame.sprite.Group()
    ship_group = pygame.sprite.Group()
    bullet_group = pygame.sprite.Group()

    # Build Level
    x = y = 0
    for row in level:
        for col in row:
            if col == "P":
                p = Platform(x, y)
                platform_group.add(p)
            x += 32
            meta_x = x
        y += 32
        x = 0

    # Set Up Camera
    total_width = len(level[0]) * 32
    total_height = len(level) * 32
    camera = Camera(total_width, total_height)
    total_rect = pygame.rect.Rect(0,0, total_width, total_height)

    # Create Game Objects
    s = Ship(total_rect)
    camera_entity = CameraEntity(total_rect)

    ship_group.add(s)
    platform_group.add(camera_entity)

    # Gameplay

    while play:
        # Checks if window exit button pressed
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_ESCAPE:
                    pygame.quit()
                    sys.exit()

        # Update
        screen.fill(WHITE)
        ship_group.update(camera_entity, bullet_group)
        camera.update(camera_entity.rect, s.rect)
        platform_group.update(camera_entity)
        bullet_group.update(s)

        # Print Everything
        for p in platform_group:
            screen.blit(p.image, camera.apply(p))

        for s in ship_group:
            screen.blit(s.image, camera.apply(s))

        for bullet in bullet_group:
            screen.blit(bullet.image, camera.apply(bullet))

        # Draw Everything
        #platform_group.draw(screen)
        #ship_group.draw(screen)

        # Limits frames per iteration of while loop
        clock.tick(fps)

        # Writes to main surface
        pygame.display.flip()

if __name__ == "__main__":
    main()
