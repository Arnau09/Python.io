# Joc de naus.io
    # =============================================================================
    # JOC DELS METEORITS
    # =============================================================================
    
    import random
    import pygame
    from pygame.locals import *


    # =============================================================================
    # CLASSE METEOR
    # =============================================================================
    class Meteor:
        def __init__(self, imatge, velocitat, pos_x, pos_y):
            self.imatge = imatge
            self.velocitat = velocitat
            self.x = pos_x
            self.y = pos_y
            self.img_meteor = pygame.image.load(self.imatge)
            self.rect_meteor = self.img_meteor.get_rect(midbottom=(self.x, self.y))

    def reiniciar(self):
        self.x = random.randint(30, 610)
        self.y = -64
        self.velocitat = random.randint(2, 10)

    def moure(self):
        self.y += self.velocitat
        self.rect_meteor = self.img_meteor.get_rect(midbottom=(self.x, self.y))

    def ha_sortit_per_baix(self):
        return self.y >= 480

    def puntuar_i_reiniciar(self):
        if self.ha_sortit_per_baix():
            self.reiniciar()
            return 5
        return 0


    # =============================================================================
    # CLASSE ESCUT CAIENT (El power-up que cau del cel)
    # =============================================================================
    class EscutCaient:
        def __init__(self, imatge):
            self.imatge = imatge
            self.img = pygame.image.load(self.imatge)
            self.reiniciar()

    def reiniciar(self):
        self.x = random.randint(30, 610)
        # Ho posem molt amunt (y negatiu) perquè trigui una mica a caure
        self.y = random.randint(-3000, -1000)
        self.velocitat = random.randint(3, 6)
        self.rect = self.img.get_rect(midbottom=(self.x, self.y))

    def moure(self):
        self.y += self.velocitat
        self.rect = self.img.get_rect(midbottom=(self.x, self.y))

    def ha_sortit_per_baix(self):
        return self.y >= 480


    # =============================================================================
    # CLASSE NAU
    # =============================================================================
    class Nau:
        def __init__(self, imatge, velocitat, pos_x, pos_y, vides, imatge_vida, imatge_escut):
            self.imatge = imatge
            self.velocitat = velocitat
            self.velocitat_normal = velocitat * 2
            self.x = pos_x
            self.y = pos_y
            self.img = pygame.image.load(self.imatge)
            self.img_vida = pygame.image.load(imatge_vida)
            self.rect = self.img.get_rect(midbottom=(self.x, self.y))
            self.vides = vides
            self.vides_originals = vides
            self.img_escut = pygame.image.load(imatge_escut)


        # Noves variables per l'escut
        self.escut_actiu = False
        self.fi_escut = 0
        self.img_escut = pygame.image.load('assets/Escut.png')

        self.turbo = False
        self.fi_turbo = 0

    def reiniciar(self):
        self.x = 300
        self.y = 460
        self.rect = self.img.get_rect(midbottom=(self.x, self.y))
        self.vides = self.vides_originals
        self.escut_actiu = False

    def moure(self, pantalla_rect):
        keys = pygame.key.get_pressed()
        if keys[K_LSHIFT] or keys[K_RSHIFT]:
            self.turbo = True
            self.fi_turbo = pygame.time.get_ticks() + 2000
        elif self.turbo and pygame.time.get_ticks() < self.fi_turbo:
            self.velocitat = 10
        else:
            self.turbo = False
            self.velocitat = 5

        if keys[K_a] or keys[K_LEFT]:
            self.rect.x -= self.velocitat
        if keys[K_d] or keys[K_RIGHT]:
            self.rect.x += self.velocitat
        if keys[K_w] or keys[K_UP]:
            self.rect.y -= self.velocitat
        if keys[K_s] or keys[K_DOWN]:
            self.rect.y += self.velocitat


        self.rect.clamp_ip(pantalla_rect)

    def activar_escut(self):
        """Activa l'escut i guarda en quin moment s'ha de desactivar (d'aquí a 2 segons)."""
        self.escut_actiu = True
        # pygame.time.get_ticks() ens dóna el temps actual en milisegons. Sumem 2000 ms (2 segons).
        self.fi_escut = pygame.time.get_ticks() + 2000

    def gestionar_escut(self):
        """Comprova si el temps de l'escut ja s'ha acabat per apagar-lo."""
        if self.escut_actiu and pygame.time.get_ticks() > self.fi_escut:
            self.escut_actiu = False

    def restar_vida(self):
        self.vides -= 1

    def esta_viva(self):
        return self.vides > 0


    # =============================================================================
    # CLASSE DISPAR
    # =============================================================================
    class Dispar:
        def __init__(self, imatge, velocitat, pos_x, pos_y):
            self.imatge = imatge
            self.velocitat = velocitat
            self.x = pos_x
            self.y = pos_y
            self.img_dispar = pygame.image.load(self.imatge)
            self.rect_dispar = self.img_dispar.get_rect(midbottom=(self.x, self.y))

    def moure(self):
        self.y -= self.velocitat
        self.rect_dispar = self.img_dispar.get_rect(midbottom=(self.x, self.y))

    def ha_sortit_per_sobre(self):
        return self.y <= 0


    # =============================================================================
    # CLASSE JOC
    # =============================================================================
    class Joc:
        PANTALLA_INICI = "inici"
        PANTALLA_JOC = "joc"
        PANTALLA_GAME_OVER = "game_over"

    def __init__(self, ample, alt, fps, nombre_meteors):
        pygame.init()
        pygame.mixer.music.load('assets/fondo.mp3')
        pygame.mixer.music.set_volume(0.5)
        self.ample = ample
        self.alt = alt
        self.fps = fps
        self.numero_meteors = nombre_meteors
        self.pantalla = pygame.display.set_mode((self.ample, self.alt))
        pygame.display.set_caption("Joc dels Meteorits")
        self.rellotge = pygame.time.Clock()

        self.meteors = []
        self.dispars = []
        self.punts = 0

        self.nau = Nau('assets/nau1.png', velocitat=5, pos_x=300, pos_y=450,
                       vides=3, imatge_vida='assets/cor.png', imatge_escut='assets/Escut.png')

        # Creem el power-up de l'escut
        self.item_escut = EscutCaient('assets/Escut.png')

        self.pantalla_activa = self.PANTALLA_INICI

    def iniciar_joc(self):
        while True:
            if self.pantalla_activa == self.PANTALLA_INICI:
                self.mostrar_pantalla_inici()
            elif self.pantalla_activa == self.PANTALLA_JOC:
                self.preparar_partida()
                self.mostrar_pantalla_joc()
            elif self.pantalla_activa == self.PANTALLA_GAME_OVER:
                self.mostrar_pantalla_game_over()

    def mostrar_pantalla_inici(self):
        img_inici = pygame.image.load('assets/Començar.png')
        img_inici = pygame.transform.scale(img_inici, (self.ample, self.alt))
        self.pantalla.blit(img_inici, (0, 0))
        pygame.display.update()
        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN and event.key == K_h:
                self.pantalla_activa = self.PANTALLA_JOC

        self.rellotge.tick(self.fps)

    def mostrar_pantalla_game_over(self):
        try:
            img_game_over = pygame.image.load('assets/Game over.png')
            self.pantalla.blit(img_game_over, (0, 0))
        except:
            self.pantalla.fill((0, 0, 0))
            self._dibuixar_text("GAME OVER", mida=64, color=(255, 0, 0), x=self.ample // 2, y=160, centrat=True)

        self._dibuixar_text(f"Puntuació: {self.punts}", mida=36, color=(255, 255, 0), x=self.ample // 2, y=380,
                            centrat=True)
        self._dibuixar_text("Prem qualsevol tecla per tornar", mida=22, color=(200, 200, 200), x=self.ample // 2, y=430,
                            centrat=True)
        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                self.pantalla_activa = self.PANTALLA_INICI

        self.rellotge.tick(self.fps)

    def mostrar_pantalla_joc(self):
        while self.pantalla_activa == self.PANTALLA_JOC:
            self._gestionar_events()
            self.pantalla.fill((0, 0, 0))

            # Lògica i moviments
            self.nau.moure(self.pantalla.get_rect())
            self.nau.gestionar_escut()  # Comprova si s'ha d'apagar l'escut
            self._moure_meteors()
            self._moure_dispars()

            # Moviment del power-up de l'escut
            self.item_escut.moure()
            if self.item_escut.ha_sortit_per_baix():
                self.item_escut.reiniciar()

            # Comprovem col·lisions
            self._control_colisions()

            # Dibuixem
            self._dibuixar_meteors()
            self._dibuixar_dispars()

            # Dibuixem l'ítem de l'escut si està caient
            self.pantalla.blit(self.item_escut.img, self.item_escut.rect)

            self._dibuixar_nau()
            self._dibuixar_vides()
            self._dibuixar_punts()

            if not self.nau.esta_viva():
                self.pantalla_activa = self.PANTALLA_GAME_OVER

            pygame.display.update()
            self.rellotge.tick(self.fps)

    def preparar_partida(self):
        self.meteors.clear()
        self.dispars.clear()
        self.item_escut.reiniciar()
        for i in range(self.numero_meteors):
            meteor_nou = Meteor('assets/meteorito.png', velocitat=0, pos_x=0, pos_y=0)
            meteor_nou.reiniciar()
            self.meteors.append(meteor_nou)
        self.nau.reiniciar()
        self.punts = 0

    def _gestionar_events(self):
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == K_SPACE:
                    nou_dispar = Dispar('assets/Dispar.png', velocitat=10,
                                        pos_x=self.nau.rect.centerx,
                                        pos_y=self.nau.rect.top)
                    self.dispars.append(nou_dispar)

    def _moure_meteors(self):
        for meteor in self.meteors:
            meteor.moure()
            self.punts += meteor.puntuar_i_reiniciar()

    def _moure_dispars(self):
        for dispar in self.dispars[:]:
            dispar.moure()
            if dispar.ha_sortit_per_sobre():
                self.dispars.remove(dispar)

    def _control_colisions(self):
        # 1. Col·lisió Ítem Escut <-> Nau
        if self.item_escut.rect.colliderect(self.nau.rect):
            self.nau.activar_escut()
            self.item_escut.reiniciar()  # Torna a marxar amunt

        # 2. Col·lisió Meteorit <-> Nau
        for meteor in self.meteors:
            if meteor.rect_meteor.colliderect(self.nau.rect):
                meteor.reiniciar()
                # Només perdem vida si NO tenim l'escut actiu
                if not self.nau.escut_actiu:
                    self.nau.restar_vida()

        # 3. Col·lisió Dispar <-> Meteorit
        for dispar in self.dispars[:]:
            for meteor in self.meteors:
                if dispar.rect_dispar.colliderect(meteor.rect_meteor):
                    meteor.reiniciar()
                    self.punts += 10
                    if dispar in self.dispars:
                        self.dispars.remove(dispar)
                    break

    def _dibuixar_meteors(self):
        for meteor in self.meteors:
            self.pantalla.blit(meteor.img_meteor, meteor.rect_meteor)

    def _dibuixar_dispars(self):
        for dispar in self.dispars:
            self.pantalla.blit(dispar.img_dispar, dispar.rect_dispar)

    def _dibuixar_nau(self):
        self.pantalla.blit(self.nau.img, self.nau.rect)

        # VORAL BLAU SI L'ESCUT ESTÀ ACTIU (Feedback visual)
        if self.nau.escut_actiu:
           rect_escut = self.nau.img_escut.get_rect(center=self.nau.rect.center)
           self.pantalla.blit(self.nau.img_escut, rect_escut)

    def _dibuixar_vides(self):
        posicions_x = [500, 540, 580]
        for i in range(self.nau.vides):
            if i < len(posicions_x):
                self.pantalla.blit(self.nau.img_vida, (posicions_x[i], 20))

    def _dibuixar_punts(self):
        self._dibuixar_text(str(self.punts), mida=32, color=(255, 255, 255), x=140, y=30, centrat=False)

    def _dibuixar_text(self, text, mida, color, x, y, centrat=False):
        font = pygame.font.SysFont(None, mida)
        imatge_text = font.render(text, True, color)
        if centrat:
            rect_text = imatge_text.get_rect(center=(x, y))
            self.pantalla.blit(imatge_text, rect_text)
        else:
            self.pantalla.blit(imatge_text, (x, y))


    # =============================================================================
    # INICI DEL PROGRAMA
    # =============================================================================
    if __name__ == "__main__":
        partida = Joc(
            ample=640,
            alt=480,
            fps=60,
            nombre_meteors=4
        )
        partida.iniciar_joc()
