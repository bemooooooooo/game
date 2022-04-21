import os
import random
import pygame
import networkx as nx

pygame.font.init()

WIDTH, HEIGHT = 1080, 750
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Space Invaders")

# Load images
RED_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_red_small.png"))
GREEN_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_green_small.png"))
BLUE_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_blue_small.png"))

# Player player
YELLOW_SPACE_SHIP = pygame.image.load(os.path.join("assets", "pixel_ship_yellow.png"))

# Lasers
RED_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_red.png"))
GREEN_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_green.png"))
BLUE_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_blue.png"))
YELLOW_LASER = pygame.image.load(os.path.join("assets", "pixel_laser_yellow.png"))

# Background
BG = pygame.transform.scale(pygame.image.load(os.path.join("assets", "background-black.png")), (WIDTH, HEIGHT))

start = 'Start'
left = 'left'
right = 'right'
shoot = 'shoot'
mas = ['left','right','shoot']
edges = [(f'{start}', f'{left}', 0), (f'{start}', f'{right}', 0), (f'{start}', f'{shoot}', 0)]

tree = nx.DiGraph()
tree.add_node('Start')
tree.add_weighted_edges_from(edges)

cnt = 0
def create_graph(graph, frst, right,left,shoot):
    global cnt
    cnt += 1
    for q in graph.neighbors(frst):
        frst = q
        left, right, shoot = left+f'{cnt}', right+f'{cnt}',shoot+f'{cnt}'
        a = [(f'{frst}', 'left', 0), (f'{frst}', f'{right}', 0), (f'{frst}', f'{shoot}', 0)]
        graph.add_weighted_edges_from(a)
    if cnt<=100:
        create_graph(graph, frst, right, left, shoot)
create_graph(tree, start, right, left, shoot)


class Laser:
    def __init__(self, x, y, img):
        self.x = x
        self.y = y
        self.img = img
        self.mask = pygame.mask.from_surface(self.img)

    def draw(self, window):
        window.blit(self.img, (self.x, self.y))
    
    def return_pos_laser(self):
        return [self.x, self.y]

    def move(self, vel):
        self.y += vel

    def off_screen(self, height):
        return not (height >= self.y >= 0)

    def collision(self, obj):
        return collide(self, obj)

class Ship:
    COOLDOWN = 30

    def __init__(self, x, y, health=100):
        self.x = x
        self.y = y
        self.health = health
        self.ship_img = None
        self.laser_img = None
        self.lasers = []
        self.cool_down_counter = 0

    def draw(self, window) -> object:
        window.blit(self.ship_img, (self.x, self.y))
        for laser in self.lasers:
            laser.draw(window)

    def move_lasers(self, vel, obj, pos_lasers):
        self.cooldown()
        for laser in self.lasers:
            laser.move(vel)
            if laser.off_screen(HEIGHT):
                self.lasers.remove(laser)
                del pos_lasers[pos_lasers.index(laser.return_pos_laser)]
            elif laser.collision(obj):
                obj.health -= 10
                self.lasers.remove(laser)
                del pos_lasers[pos_lasers.index(laser.return_pos_laser)]

    def cooldown(self):
        if self.cool_down_counter >= self.COOLDOWN:
            self.cool_down_counter = 0
        elif self.cool_down_counter > 0:
            self.cool_down_counter += 1

    def shoot(self):
        if self.cool_down_counter == 0:
            laser = Laser(self.x, self.y, self.laser_img)
            self.lasers.append(laser)
            self.cool_down_counter = 1

    def get_width(self):
        return self.ship_img.get_width()

    def get_height(self):
        return self.ship_img.get_height()

class Player(Ship):
    def __init__(self, x, y, health=100):
        super().__init__(x, y, health)
        self.ship_img = YELLOW_SPACE_SHIP
        self.laser_img = YELLOW_LASER
        self.mask = pygame.mask.from_surface(self.ship_img)
        self.max_health = health

    def move_lasers(self, vel, objs):
        self.cooldown()
        for laser in self.lasers:
            laser.move(vel)
            if laser.off_screen(HEIGHT):
                self.lasers.remove(laser)
            else:
                for obj in objs:
                    if laser.collision(obj):
                        objs.remove(obj)
                        if laser in self.lasers:
                            self.lasers.remove(laser)

    def draw(self, window):
        super().draw(window)
        self.healthbar(window)

    def healthbar(self, window):
        pygame.draw.rect(window, (255, 0, 0),
                         (self.x, self.y + self.ship_img.get_height() + 10, self.ship_img.get_width(), 10))
        pygame.draw.rect(window, (0, 255, 0), (
            self.x, self.y + self.ship_img.get_height() + 10,
            self.ship_img.get_width() * (self.health / self.max_health),
            10))

    def return_pos_player(self):
        return [self.x, self.y]

class Enemy(Ship):
    COLOR_MAP = {
        "red": (RED_SPACE_SHIP, RED_LASER),
        "green": (GREEN_SPACE_SHIP, GREEN_LASER),
        "blue": (BLUE_SPACE_SHIP, BLUE_LASER)
    }

    def __init__(self, x, y, color, health=100):
        super().__init__(x, y, health)
        self.ship_img, self.laser_img = self.COLOR_MAP[color]
        self.mask = pygame.mask.from_surface(self.ship_img)

    def move(self, vel):
        self.y += vel

    def shoot(self, pos_lasers):
        if self.cool_down_counter == 0:
            laser = Laser(self.x - 20, self.y, self.laser_img)
            self.lasers.append(laser)
            self.cool_down_counter = 1
            pos_lasers.append(laser.return_pos_laser)
    
    def return_pos(self):
        return self.x, self.y

def collide(obj1, obj2):
    offset_x = obj2.x - obj1.x
    offset_y = obj2.y - obj1.y
    return obj1.mask.overlap(obj2.mask, (offset_x, offset_y)) is not None

def func(box, action, move_velocity, pos_player, pos_enemies, pos_lasers):
    kills = 0
    detetct_damage = 0
    lives_loss = 0
    if action == 'left':
        pred_pos_player = pos_player[0]-move_velocity
        for x,y in pos_lasers:
            if pred_pos_player-box[0]/2 <= x and x >= pred_pos_player+box[0]/2 and pos_player[1]+box[1]/2-move_velocity<= y and y <= pos_player[1]+box[1]/2-move_velocity: detetct_damage+=1
            #   Если лазер попадет
        for i, j in pos_enemies:
            if pred_pos_player-box[0]/2 <= i and i >= pred_pos_player+box[0]/2 and pos_player[1]+box[1]/2-move_velocity<= j and j <= pos_player[1]+box[1]/2-move_velocity: detetct_damage+=1
            #   Крч тут если в нас враг врежется
            if j<=pos_player[1]+move_velocity:lives_loss+=1
            #   Если враг елетит за экран
    if action == 'right':
        pred_pos_player = pos_player[0]+move_velocity
        for x,y in pos_lasers:
            if pred_pos_player-box[0]/2 <= x and x >= pred_pos_player+box[0]/2 and pos_player[1]+box[1]/2+move_velocity<= y and y <= pos_player[1]+box[1]/2+move_velocity: detetct_damage+=1
            #   Если лазер попадет
        for i, j in pos_enemies:
            if pred_pos_player-box[0]/2 <= i and i >= pred_pos_player+box[0]/2 and pos_player[1]+box[1]/2+move_velocity<= j and j <= pos_player[1]+box[1]/2+move_velocity: detetct_damage+=1
            #   Крч тут если в нас враг врежется
            if j<=pos_player[1]+move_velocity:lives_loss+=1
            #   Если враг вылетит за экран   
    if action == 'shoot':
        for i, j in pos_enemies: 
            if i-5 <=pos_player[0] and pos_player[0] <= i+5: kills+=1
    return kills-0.1*detetct_damage-0.2*lives_loss

def II(level, graph, box, move_velocity, pos_player, pos_enemies, pos_lasers):
    global mas
    for i in range(101):
        for w in mas:
            for e in mas:
                graph.add_weighted_edges_from([(f'{w}'+str(i),f'{e}'+str(i), func(box, w, move_velocity, pos_player, pos_enemies[level-1], pos_lasers))])
    ret = nx.dag_longest_path(graph, weight='weight')[1]
    return ret

def main():
    run = True
    fps = 60
    level = 0
    lives = 5
    main_font = pygame.font.SysFont("comicsans", 50)
    lost_font = pygame.font.SysFont("comicsans", 60)

    enemies = []
    enemies_pos = [] 
    wave_length = 5
    enemy_vel = 1
    pos_lasers = []

    player_vel = 5
    laser_vel = 5

    box_player = [300,630]
    player = Player(300, 630)

    clock = pygame.time.Clock()
    time = clock.get_time()

    lost = False
    lost_count = 0

    def redraw_window():
        WIN.blit(BG, (0, 0))
        lives_label = main_font.render(f"Lives: {lives}", 1, (255, 255, 255))
        level_label = main_font.render(f"Level: {level}", 1, (255, 255, 255))

        WIN.blit(lives_label, (10, 10))
        WIN.blit(level_label, (WIDTH - level_label.get_width() - 10, 10))

        for enemy in enemies:
            enemy.draw(WIN)

        player.draw(WIN)

        if lost:
            lost_label = lost_font.render("You Lost!!", 1, (255, 255, 255))
            WIN.blit(lost_label, (WIDTH / 2 - lost_label.get_width() / 2, 350))

        pygame.display.update()

    while run:
        clock.tick(fps)
        redraw_window()

        if lives <= 0 or player.health <= 0:
            lost = True
            lost_count += 1

        if lost:
            if lost_count > fps * 3:
                run = False
            else:
                continue

        if len(enemies) == 0:
            level += 1
            enemies_pos.append([])
            wave_length += 5
            for i in range(wave_length):
                a,b = random.randrange(50, WIDTH - 100), random.randrange(-1500, -100)
                enemy = Enemy(a,b,random.choice(["red", "blue", "green"]))
                enemies.append(enemy)
                enemies_pos[level-1].append(enemy.return_pos())
                
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                quit()

        if time%10 == 0:
            action = II(level,tree,box_player, player_vel, [player.x, player.y], enemies_pos, pos_lasers)
            if action == 'left' and player.x-player_vel>0:player.x-=player_vel
            if action == 'right' and player.x+player_vel>0:player.x+=player_vel
            if action == 'shoot':player.shoot()
         
#        keys = pygame.key.get_pressed()
#        if keys[pygame.K_a] and player.x - player_vel > 0:  # влево
#            player.x -= player_vel
#        if keys[pygame.K_d] and player.x + player_vel + player.get_width() < WIDTH:  # вправо
#            player.x += player_vel
#        if keys[pygame.K_w] and player.y - player_vel > 0:  # вверх
#            player.y -= player_vel
#        if keys[pygame.K_s] and player.y + player_vel + player.get_height() + 15 < HEIGHT:  # вниз
#            player.y += player_vel
#        if keys[pygame.K_SPACE]:                    # стрельба
#            player.shoot()
 
        for enemy in enemies[:]:
            enemy.move(enemy_vel)
            enemy.move_lasers(laser_vel, player, pos_lasers)

            if random.randrange(0, 2*60) == 1:
                enemy.shoot(pos_lasers)

            if collide(enemy, player):
                player.health -= 10
                enemies.remove(enemy)
            elif enemy.y + enemy.get_height() > HEIGHT:
                lives -= 1
                enemies.remove(enemy)

        player.move_lasers(-laser_vel, enemies)

def main_menu():
    title_font = pygame.font.SysFont("comicsans", 70)
    run = True
    while run:
        WIN.blit(BG, (0, 0))
        title_label = title_font.render("Press the mouse to begin...", 1, (255, 255, 255))
        WIN.blit(title_label, (WIDTH / 2 - title_label.get_width() / 2, 350))
        pygame.display.update()
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False
            if event.type == pygame.MOUSEBUTTONDOWN:
                main()
    pygame.quit()

main_menu()
