import pygame
import time
from random import randint

##########################
#    INISIASI PROGRAM    #
##########################

#inisiasi library pygame
pygame.init()

#definisi warna-warna
green = (0, 255,0)
cyan = (0, 255, 255)
blue = (0, 0, 255)
magenta = (255,0,255)
red = (255, 0, 0)
yellow = (255,255,0)
black = (0,0,0)
white = (192,192,192)
grey = (64,64,64)

#bikin layar
screen_size = (800, 600)
screen = pygame.display.set_mode(screen_size)
pygame.display.set_caption('Game Osiris')

#menyiapkan game assetsd

#bikin gambar bata warna merah
bata_merah = pygame.Surface((60,20))
bata_merah.fill(red)

#bikin gambar bata warna kuninhg
bata_kuning = pygame.Surface((60,20))
bata_kuning.fill(yellow)

#bikin gambar stick warna cyan
stick_cyan = pygame.Surface((64,8))
stick_cyan.fill(cyan)

#bikin gambar bola berbentuk lingkaran warna hijau
bola_hijau = pygame.Surface((16,16))
bola_hijau.fill(black)
pygame.draw.circle(bola_hijau,green,(8,8),8)
bola_hijau.set_colorkey(black)


################################
#    DEFINISI CLASS 'Benda'    #
################################

class Benda:
    # atribut benda yang kita definiskan adalah lokasi x,y, ukuran lebar dan tingginya, serta gambarnya
    def __init__(self, x, y, w, h, gambar):
        self.x = x
        self.y = y
        self.w = w
        self.h = h
        self.gambar = gambar
      
    # memeriksa tabrakan antara benda ini dengan kumpulan benda lain 
    def cek_tabrakan(self,daftar_benda):
        tembolok = []
        #tentukan benda-benda yang tabrakan, masukkan ke list tembolok
        for bata in daftar_benda:
            if (bata.x > self.x + self.w):
                pass
            elif (bata.x + bata.w < self.x):
                pass
            elif (bata.y > self.y + self.h):
                pass
            elif (bata.y + bata.h < self.y):
                pass
            else:
                tembolok.append(bata)

        #tentukan arah tabrakan dari benda-benda yang sudah tercatat di tembolok
        hasil = []
        for benda in tembolok:
            tumbukan = {"benda":benda,"v":0,"h":0}
            
            if (benda.x <= self.x and benda.x + benda.w <= self.x + self.w):
                tumbukan["h"] = -1
            elif(benda.x >= self.x and benda.x + benda.w >= self.x + self.w):
                tumbukan["h"] = 1

            if (benda.y <= self.y and benda.y + benda.h <= self.y + self.h):
                tumbukan["v"] = -1
            elif(benda.y >= self.y and benda.y + benda.h >= self.y + self.h):
                tumbukan["v"] = 1
            hasil.append(tumbukan)
            
        return hasil

############################
#    OUTER PROGRAM LOOP    #
############################
while True:
    #######################
    #   WELCOME SCREEN    #
    #######################

    #tampilkan layar selamat datang
    screen.fill(grey)
    font = pygame.font.SysFont("comicsansms", 36)
    message = "SELAMAT DATANG"
    text = font.render(message, True, blue, None)
    textRect = text.get_rect()  
    textRect.center = (screen_size[0]// 2, screen_size[1] // 2) 
    screen.blit(text, textRect) 

    message = "tekan enter untuk mulai, esc untuk keluar"
    text = font.render(message, True, green, None)
    textRect = text.get_rect()  
    textRect.center = (screen_size[0]// 2, screen_size[1] // 2 + 100) 
    screen.blit(text, textRect) 

    pygame.display.flip()

    #welcome screen loop : tunggu input pengguna
    mulai = False
    while not mulai:
        #AMBIL EVENT DAN AKSI PENGGUNA
        evs = pygame.event.get()
        pressed = pygame.key.get_pressed()
        for ev in evs:
            if ev.type == pygame.QUIT :
                pygame.quit()
                exit()
            elif ev.type == pygame.KEYDOWN :
                if ev.key ==pygame.K_ESCAPE :
                    pygame.quit()
                    exit()
                elif ev.key ==pygame.K_RETURN :
                    mulai = True

    ### PROGRAM UTAMA GAME ###
    ####################################
    #    INISIASI KONDISI AWAL GAME    #
    ####################################
    gameover = False
    gamestate = ""

    
    #bikin tembok, list yang akan menampung daftar bata di game ini 
    tembok  = []
    
    #bikin 5 row bata
    for i in [0,1,2,3,4]:
        #kita bikin selang-seling warna batanya
        if (i % 2 == 0):
            gbr = bata_kuning
        else:
            gbr = bata_merah
        
        #dalam 1 row ada 8 bata
        for j in range(0,8):
            #bikin objek dari class 'Benda'
            bata = Benda(j*80+80, i*40+80, 60, 20 , gbr)
            #lalu daftarkan ke list tembok
            tembok.append(bata)

    #bikin bola
    #posisi awal x diacak antara 300 s/d 500
    x = randint(300,500)
    
    #posisi awal y = 300
    y = 300
    
    ball = Benda(x, y , 16 , 16 , bola_hijau)
    
    #tentukan arah gerakan awal bola
    #arah x kita acak antara -1 atau 1
    ball_move_x = (-1,1)[randint(0,1)]
    #arah y ke atas
    ball_move_y = -1
    
    #tentukan kecepatan bola
    ball_speed = 40 
    
    #bikin stick
    stick = Benda(384,450, 64, 8, stick_cyan)
    
    #tentukan batas pergerakan stick
    stick_range = (0,400,732,482)
    
    #tentukan kecepatan stick
    stick_speed = 48
    
    #tentukan warna begron layar
    background = black
    
    #pencatatan awal timer
    clk = pygame.time.Clock()
    last_timer_tick = pygame.time.get_ticks()

    ###################
    #    GAME LOOP    #
    ###################
    while not gameover :        
        #reset arah gerakan stick
        stick_movex = 0
        stick_movey = 0
        

        ##############################
        #    AMBIL INPUT PENGGUNA    #
        ##############################
        evs = pygame.event.get()
        pressed = pygame.key.get_pressed()
        for ev in evs:
            if ev.type == pygame.QUIT :
                gameover = True
            elif ev.type == pygame.KEYDOWN :
                if ev.key ==pygame.K_ESCAPE :
                    gameover = True
                    
        #CHECK TOMBOL KIRI KANAN
        if (pressed[pygame.K_d]):
            #kalau user pencet tombol 'd', arah gerakan stick x positif
            stick_movex = 1 
            
        elif (pressed[pygame.K_a]):
            #kalau user pencet tombol 'a', arah gerakan stick x negatif
            stick_movex = -1

        #CHECK TOMBOL ATAS BAWAH
        if (pressed[pygame.K_w]):
            #kalau user pencet tombol 'w', arah gerakan stick y negatif
            stick_movey = -1
        elif (pressed[pygame.K_s]):
            #kalau user pencet tombol 'w', arah gerakan stick y positif
            stick_movey = 1
        
        
                    
        #############################
        #    UPDATE KONDISI GAME    #
        #############################
        
        #ambil clock tick saat ini
        current_timer_tick = pygame.time.get_ticks()
        
        #hitung delta waktu antara perulangan sebelumnya dengan perulangan sekarang
        elapsed_time = current_timer_tick - last_timer_tick
        
        #update catatan waktu sebelumnya dengan waktu sekarang
        last_timer_tick = current_timer_tick
        
        #tentukan besar pergerakan stick berdasarkan kecepatan, arah, dan delta waktu
        delta_x = float(stick_movex * stick_speed * elapsed_time) / 110
        delta_y = float(stick_movey * stick_speed * elapsed_time) / 110
        
        #update posisi stick
        stick.x += delta_x
        stick.y += delta_y

        #pastikan posisi stik masih berada di range gerakan yang sudah kita tentukan sebelumnya
        if (stick.x < stick_range[0]):
            stick.x = stick_range[0]
        elif (stick.x > stick_range[2]):
            stick.x = stick_range[2]
        
        if (stick.y < stick_range[1]):
            stick.y = stick_range[1]
        elif (stick.y > stick_range[3]):
            stick.y = stick_range[3]
        
        
        #tentukan besar pergerakan bola berdasarkan kecepatan, arah, dan delta waktu
        delta_x = float(ball_move_x * ball_speed * elapsed_time) / 200
        delta_y = float(ball_move_y * ball_speed * elapsed_time) / 200

        #update posisi bola
        ball.x += delta_x
        ball.y += delta_y

        #cek apakah bola menabrak sisi-sisi layar, pantulkan bola kalau iya
        if (ball.x < 0):
            ball.x = 0
            ball_move_x = 1
        elif (ball.x > 784):
            ball.x = 784
            ball_move_x = -1
        
        if (ball.y < 0):
            ball.y = 0
            ball_move_y = 1
            
        #kalau bola menyentuh dasar layar, player kalah
        if (ball.y > 500):
            gamestate = "KALAH"
            gameover = True
        else:
            #kalau tidak, cek apakah bola bertumbukan dengan bata-bata di tembok
            tabrakans = ball.cek_tabrakan(tembok)
            for tumbukan in tabrakans:
                v = tumbukan["v"]
                h = tumbukan["h"]
                benda = tumbukan["benda"]

                if (v == 1):
                    ball_move_y = -1
                elif (v == -1):
                    ball_move_y = 1
                
                if (h == 1):
                    ball_move_x = -1
                elif (h == -1):
                    ball_move_x = 1
                
                #hapus bata yang ketabrak bola dari list tembok
                tembok.remove(benda)
                    
                    
            #kalau tembok sudah kosong, player menang
            if (len(tembok) == 0):
                gamestate = "MENANG"
                gameover = True
            else:
                #kalau belum menang, cek apakah bola bertumbukan dengan stick
                tabrakans = ball.cek_tabrakan([stick])
                for tumbukan in tabrakans:
                    v = tumbukan["v"]
                    h = tumbukan["h"]
                    benda = tumbukan["benda"]
                    ball_move_y = -1
                

        ##############################
        #    TAMPILKAN LAYAR GAME    #
        ##############################
        #kosongkan layar dengan warna background
        screen.fill(background)
        
        #tampilkan semua bata yang ada di list tembok 
        for bata in tembok:
            #tempel gambar bata ke layar, sesuai gambar yang tercatat di object bata tersebut dan lokasi x,y-nya
            screen.blit(bata.gambar,(bata.x,bata.y))

        #tempel gambar bola ke layar sesuai lokasi x,y-nya
        screen.blit(ball.gambar,(ball.x,ball.y))
        
        #tempel gambar stick ke layar sesuai lokasi x,y-nya
        screen.blit(stick.gambar,(stick.x,stick.y))
        
        #tampilkan gambar ke layar
        pygame.display.flip()
        
        #batasi refresh layar paling banyak 60 FPS
        clk.tick(60)

    #######################
    #    ENDING SCREEN    #
    #######################
    if "MENANG" == gamestate :
        message = "KAMU MENANG !!!"
    elif "KALAH" == gamestate :
        message = "KAMU KALAH !!!"
    else :
        message = "GAME OVER !!!"
        
    #kosongkan layar dengan warna grey
    screen.fill(grey)
    
    #bikin gambar text
    font = pygame.font.SysFont("comicsansms", 36)
    text = font.render(message, True, red, None)
    textRect = text.get_rect()  
    textRect.center = (screen_size[0]// 2, screen_size[1] // 2) 
    
    #tempel gambar text ke layar
    screen.blit(text, textRect) 
    
    #tampilkan  layar
    pygame.display.flip()
    
    #kasih jeda 3 detik sebelum kembali ke layar selamat datang
    time.sleep(3)
