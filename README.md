# poppyplay.shoter
from pygame import *
from random import randint

font.init()
mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')



#нам нужны такие картинки:
img_back = "doy.jpg" #фон игры
img_hero = "Wounded_Agressive_Huggy_Wuggy.webp" #герой
img_enemy = "ppt.png"

#класс-родитель для других спрайтов
class GameSprite(sprite.Sprite):
 #конструктор класса
   def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
       #Вызываем конструктор класса (Sprite):
       sprite.Sprite.__init__(self)


       #каждый спрайт должен хранить свойство image - изображение
       self.image = transform.scale(image.load(player_image), (size_x, size_y))
       self.speed = player_speed


       #каждый спрайт должен хранить свойство rect - прямоугольник, в который он вписан
       self.rect = self.image.get_rect()
       self.rect.x = player_x
       self.rect.y = player_y
 #метод, отрисовывающий героя на окне
   def reset(self):
       window.blit(self.image, (self.rect.x, self.rect.y))


#класс главного игрока
class Player(GameSprite):
   #метод для управления спрайтом стрелками клавиатуры
   def update(self):
       keys = key.get_pressed()
       if keys[K_LEFT] and self.rect.x > 5:
           self.rect.x -= self.speed
       if keys[K_RIGHT] and self.rect.x < win_width - 80:
           self.rect.x += self.speed
 #метод "выстрел" (используем место игрока, чтобы создать там пулю)
   def fire(self):
       bullet = Bullet('poppy.png', self.rect.centerx, self.rect.top, 15, 20, -15)
       bullets.add(bullet)



class Enamy(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y > win_height:
            self.rect.x = randint(80, win_height - 80)
            self.rect.y = 0
        
class Bullet(GameSprite):
    def update(self):
        self.rect.y += self.speed
        if self.rect.y < 0:
            self.kill()     

#Создаем окошко
win_width = 700
win_height = 500
display.set_caption("Shooter")
window = display.set_mode((win_width, win_height))
background = transform.scale(image.load(img_back), (win_width, win_height))
score = 0
lost = 0
font1 = font.Font(None, 36)
font2 = font.Font(None, 36)

 

#создаем спрайты
ship = Player(img_hero, 5, win_height - 100, 80, 100, 10)

bullets = sprite.Group()
ufos = sprite.Group()
for i in range(5):
    ufo = Enamy(img_enemy, 5, randint(50, 650), 80, 100, randint(1,5))
    ufos.add(ufo)

#переменная "игра закончилась": как только там True, в основном цикле перестают работать спрайты
finish = False
#Основной цикл игры:
run = True #флаг сбрасывается кнопкой закрытия окна
while run:
   #событие нажатия на кнопку Закрыть
   for e in event.get():
       if e.type == QUIT:
           run = False
       elif e.type == KEYDOWN:
           if e.key == K_SPACE:
               ship.fire()


   if not finish:
       t2= font2.render('Счёт:' + str(score), 1, (255, 255, 255))
       t1= font1.render('Пропущено:' + str(lost), 1, (255, 255, 255))
       #обновляем фон
       window.blit(background,(0,0))


       #производим движения спрайтов
       ship.update()
       ufos.update()
       bullets.update()
       window.blit(t1, (10, 20))


       #обновляем их в новом местоположении при каждой итерации цикла
       ship.reset()
       ufos.draw(window)
       bullets.draw(window)
       collides = sprite.groupcollide(ufos, bullets, True, True)
       for c in collides:
           score = score +1
           ufo = Enamy(img_enemy, 5, randint(50, 650), 80, 100, randint(1,5))
           ufos.add(ufo)

       display.update()
   #цикл срабатывает каждые 0.05 секунд
   time.delay(50)

