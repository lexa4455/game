import pygame
import sys
import random

clock = pygame.time.Clock()

# Инициализация Pygame
pygame.init()

# Определение размеров окна
WIDTH, HEIGHT = 1200, 675
MAX_FPS = 60
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Secret Order")

# Загрузка иконки
icon = pygame.image.load("images/icon.png").convert_alpha()
pygame.display.set_icon(icon)

# Загрузка фона
bg = pygame.image.load("bg game/bg1200x675.png").convert_alpha()

bg_sound = pygame.mixer.Sound('sounds/bg.mp3')
bg_sound.play()

# Размеры спрайтов игрока
PLAYER_WIDTH, PLAYER_HEIGHT = 64, 64

life = pygame.image.load("images/hp.png").convert_alpha()

# Анимации персонажа
walk_left = [
    pygame.transform.scale(pygame.image.load("images/player_left/player_left1.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/player_left/player_left2.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/player_left/player_left3.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/player_left/player_left4.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
]
walk_right = [
    pygame.transform.scale(pygame.image.load("images/player_right/player_right1.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/player_right/player_right2.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/player_right/player_right3.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/player_right/player_right4.png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
]

player_anim_count = 0
bg_x = 0

# Загрузка анимации врагов
robot_anim1 = [
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim (1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 2 (1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 3(1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 4 (1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 5 (1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 6 (1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 7(1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim1/Robots anim 8(1).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
]

robot_anim2 = [
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim1 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim2 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim3 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim4 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim5 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim6 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim7 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
    pygame.transform.scale(pygame.image.load("images/robotanim2/Robots anim8 (2).png").convert_alpha(), (PLAYER_WIDTH, PLAYER_HEIGHT)),
]

# Класс для пуль
class Bullet:
    def __init__(self, x, y, direction, speed=10, is_enemy_bullet=False):
        self.x = x
        self.y = y
        self.speed = speed
        self.direction = direction
        self.image = pygame.transform.scale(pygame.image.load("images/bullet2.png").convert_alpha(), (20, 10))
        self.is_enemy_bullet = is_enemy_bullet

    def move(self):
        if self.direction == 'right':
            self.x += self.speed
        else:
            self.x -= self.speed

    def draw(self, screen):
        screen.blit(self.image, (self.x, self.y))

    def off_screen(self):
        return self.x < 0 or self.x > WIDTH

    def hit_enemy(self, enemy):
        return enemy.x < self.x < enemy.x + PLAYER_WIDTH and enemy.y < self.y < enemy.y + PLAYER_HEIGHT

    def hit_player(self, player_x, player_y):
        return player_x < self.x < player_x + PLAYER_WIDTH and player_y < self.y < player_y + PLAYER_HEIGHT

# Класс врага типа 1
class EnemyType1:
    def __init__(self):
        self.x = WIDTH + random.randint(50, 300)
        self.y = 590
        self.speed = 5
        self.anim_count = 0
        self.hp = 3  # Количество жизней врага

    def move(self):
        self.x -= self.speed

    def draw(self, screen):
        if self.anim_count >= len(robot_anim1):
            self.anim_count = 0
        screen.blit(robot_anim1[self.anim_count], (self.x, self.y))
        self.anim_count += 1

    def off_screen(self):
        return self.x < -50

    def hit(self):
        self.hp -= 1
        return self.hp <= 0

# Класс врага типа 2
class EnemyType2:
    def __init__(self):
        self.x = WIDTH + random.randint(50, 300)
        self.y = 590
        self.speed = 7
        self.anim_count = 0
        self.hp = 2  # У второго врага меньше здоровья
        self.shoot_timer = 0  # Таймер для стрельбы

    def move(self):
        self.x -= self.speed

    def draw(self, screen):
        if self.anim_count >= len(robot_anim2):
            self.anim_count = 0
        screen.blit(robot_anim2[self.anim_count], (self.x, self.y))
        self.anim_count += 1

    def off_screen(self):
        return self.x < -50

    def hit(self):
        self.hp -= 1
        return self.hp <= 0

    def shoot(self, bullets):
        if self.shoot_timer <= 0:  # Если прошло достаточно времени для стрельбы
            bullet_x = self.x
            bullet_y = self.y + PLAYER_HEIGHT // 2
            bullets.append(Bullet(bullet_x, bullet_y, 'left', speed=12, is_enemy_bullet=True))
            self.shoot_timer = 60  # Таймер стрельбы
        else:
            self.shoot_timer -= 1

# Начальные параметры
bg_speed = 2
player_speed = 5
player_x = 100
player_y = 590
player_direction = 'right'
is_jumping = False
jump_speed = 15
gravity = 1
vertical_velocity = 0
bullets = []  # Список для пуль
ammo = 10  # Количество патронов
reload_time = 2000  # Время перезарядки в миллисекундах
last_reload_time = 0  # Время последней перезарядки
is_reloading = False  # Флаг перезарядки
player_hp = 5  # Количество жизней игрока

LEFT_BOUNDARY = 50
RIGHT_BOUNDARY = WIDTH - 500

animation_speed = 5
animation_counter = 0

# Список врагов
enemies = []
spawn_delay = 200
enemy_timer = 10

# Шрифт для отображения патронов и жизней
font = pygame.font.SysFont(None, 36)

# Основной игровой цикл
running = True
while running:
    clock.tick(MAX_FPS)

    # Обработка событий
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
            pygame.quit()
            sys.exit()

        # Стрельба
        if event.type == pygame.MOUSEBUTTONDOWN and ammo > 0 and not is_reloading:
            if event.button == 1:  # Левая кнопка мыши
                bullet_x = player_x + (PLAYER_WIDTH if player_direction == 'right' else 0)
                bullet_y = player_y + PLAYER_HEIGHT // 2
                bullets.append(Bullet(bullet_x, bullet_y, player_direction))
                ammo -= 1  # Уменьшаем количество патронов

        # Начало перезарядки
        if event.type == pygame.KEYDOWN and ammo == 0 and not is_reloading:
            is_reloading = True
            last_reload_time = pygame.time.get_ticks()  # Сохраняем текущее время

    # Управление персонажем
    keys = pygame.key.get_pressed()
    if keys[pygame.K_a]:
        player_x -= player_speed
        player_direction = 'left'
    if keys[pygame.K_d]:
        player_x += player_speed
        player_direction = 'right'

    if player_x < LEFT_BOUNDARY:
        player_x = LEFT_BOUNDARY
    if player_x > RIGHT_BOUNDARY:
        player_x = RIGHT_BOUNDARY

    # Прыжок
    if not is_jumping:
        if keys[pygame.K_SPACE]:
            is_jumping = True
            vertical_velocity = -jump_speed
    else:
        player_y += vertical_velocity
        vertical_velocity += gravity
        if player_y >= 590:
            player_y = 590
            is_jumping = False

    # Движение фона
    bg_x -= bg_speed
    if bg_x <= -1200:
        bg_x = 0

    # Спавн врагов
    enemy_timer += 1
    if enemy_timer >= spawn_delay:
        if random.randint(0, 1) == 0:
            enemies.append(EnemyType1())
        else:
            enemies.append(EnemyType2())
        enemy_timer = 0

    # Движение врагов
    for enemy in enemies[:]:
        enemy.move()
        if isinstance(enemy, EnemyType2):  # Враг типа 2 стреляет
            enemy.shoot(bullets)
        if enemy.off_screen():
            enemies.remove(enemy)

    # Движение пуль
    for bullet in bullets[:]:
        bullet.move()
        if bullet.off_screen():
            bullets.remove(bullet)

    # Проверка попадания пули во врага
    for bullet in bullets[:]:
        if not bullet.is_enemy_bullet:  # Пули игрока
            for enemy in enemies[:]:
                if bullet.hit_enemy(enemy):
                    if enemy.hit():
                        enemies.remove(enemy)
                    bullets.remove(bullet)
                    break

    # Проверка столкновений врага с игроком
    for enemy in enemies[:]:
        if enemy.x < player_x + PLAYER_WIDTH and enemy.x + PLAYER_WIDTH > player_x and enemy.y < player_y + PLAYER_HEIGHT and enemy.y + PLAYER_HEIGHT > player_y:
            player_hp -= 1  # Уменьшаем здоровье игрока при столкновении
            enemies.remove(enemy)

    # Проверка попадания в игрока пули врага
    for bullet in bullets[:]:
        if bullet.is_enemy_bullet and bullet.hit_player(player_x, player_y):
            player_hp -= 1  # Уменьшаем здоровье игрока при попадании
            bullets.remove(bullet)

    # Обработка перезарядки
    if is_reloading:
        current_time = pygame.time.get_ticks()
        if current_time - last_reload_time >= reload_time:
            ammo = 10  # Восстанавливаем патроны
            is_reloading = False

    # Отрисовка фона
    screen.blit(bg, (bg_x, 0))
    screen.blit(bg, (bg_x + 1200, 0))

    # Анимация персонажа
    animation_counter += 1
    if animation_counter >= animation_speed:
        player_anim_count += 1
        animation_counter = 0

    if player_anim_count >= len(walk_right):
        player_anim_count = 0

    # Отрисовка персонажа
    if player_direction == 'right':
        screen.blit(walk_right[player_anim_count], (player_x, player_y))
    else:
        screen.blit(walk_left[player_anim_count], (player_x, player_y))

    # Отрисовка врагов
    for enemy in enemies:
        enemy.draw(screen)

    # Отрисовка пуль
    for bullet in bullets:
        bullet.draw(screen)

    # Отображение количества патронов и жизней игрока
    ammo_text = font.render(f"Ammo: {ammo}", True, (255, 255, 255))
    hp_text = font.render(f"HP: {player_hp}", True, (255, 0, 0))
    reload_text = font.render(f"Reloading..." if is_reloading else "", True, (255, 255, 255))
    screen.blit(ammo_text, (10, 10))
    screen.blit(hp_text, (10, 50))
    screen.blit(reload_text, (10, 90))  # Показать текст перезарядки

    # Обновление экрана
    pygame.display.update()
