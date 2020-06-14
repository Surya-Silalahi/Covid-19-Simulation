import random
import pygame
import math


class Person():
    #status "Healthy", "sick", "recovered"
    def __init__(self, x, y, status, socialDistancing):
        self.colors = {"healthy": "green", "sick": "red", "recovered": "blue", "dead": "black"}
        self.x = x
        self.y = y
        self.status = status
        self.socialDistancing = socialDistancing
        self.radius = 6
        self.vx = 0
        self.vy = 0
        self.turnSick = 0
        self.recoveryTime = random.randint(400, 430)
        self.deadTime = random.randint(410, 430)

        while -0.5 < self.vx < 0.5 and -0.5 < self.vy < 0.5:
            self.vx = random.uniform(-5, 5)
            self.vy = random.uniform(-5, 5)

    #screen the main surface
    def draw(self, screen):
        pygame.draw.circle(screen, pygame.Color(self.colors[self.status]), (round(self.x), round(self.y)), self.radius)


    #execute once per frame
    def update(self, screen, people):
        self.move()
        if self.status == "sick":
            self.turnSick += 1
            if self.turnSick == self.recoveryTime:
                self.status = "recovered"
                return 1
            if self.turnSick == self.deadTime:
                self.status = "dead"
                return 2


        #check for collisions
        self.checkCollidingWithWall(screen)
        for other in people:
            if self != other:       #ensure you don't check for collisions with yourself
                if self.checkCollidingWithOther(other):
                    self.updateCollisionVelocities(other)
                    #update status
                    if self.status == "sick" and other.status == "healthy":
                        other.status = "sick"
                    elif other.status == "sick" and self.status == "healthy":
                        self.status = "sick"

    def move(self):
        if not self.socialDistancing:
            self.x += random.randint(-3, 3)
            self.y += random.randint(-3, 3)

    #check for collisions with walls and update velocity
    def checkCollidingWithWall(self, screen):
        if self.x + self.radius >= screen.get_width() and self.vx > 0:      #self.vx > 0 is to prevent it from getting stuck to the wall
            self.vx *= -1
        elif self.x - self.radius <= 0 and self.vx < 0:
            self.vx *= -1
        if self.y + self.radius >= screen.get_height() and self.vy > 0:
            self.vy *= -1
        elif self.y - self.radius <= 0 and self.vy < 0:
            self.vy *= -1

    #return True if the self is colliding with other , False otherwise
    def checkCollidingWithOther(self, other):
        distance = math.sqrt(math.pow(self.x - other.x, 2) + math.pow(self.y - other.y, 2))
        if distance <= self.radius + other.radius:
            return True
        return False

    #update velocities on collision
    def updateCollisionVelocities(self, other):
        # type 1 collision - both object are moving; neither is social distancing - switch velocities
        if not self.socialDistancing and not other.socialDistancing:
            tempVX = self.vx
            tempVY = self.vy
            self.vx = other.vx
            self.vy = other.vy
            other.vx = tempVX
            other.vy = tempVY

        #type 2 collision - one object that is social distancing and one that is not
        elif other.socialDistancing:
            magV = math.sqrt(math.pow(self.vx, 2) + math.pow(self.vy, 2))
            tempVector = (self.vx + (self.x - other.x), self.vy + (self.y -other.y))
            magTempVector = math.sqrt(math.pow(tempVector[0], 2) + math.pow(tempVector[1], 2))
            normTempVector = (tempVector[0]/magTempVector, tempVector[1]/magTempVector)
            self.vx = normTempVector[0] * magV
            self.vy = normTempVector[1] * magV
WIDTH = 1000
HEIGHT = 600
screen = pygame.display.set_mode([WIDTH, HEIGHT])

def make_line(person, sick):
    person.draw(screen)
    if person.status == "sick":
        sick.append((person))
        for x in range(len(sick)):
            pygame.draw.line(screen, pygame.Color("yellow"), (int(sick[len(sick) - 1].x), int(sick[len(sick) - 1].y)),(int(sick[len(sick) - 2].x), int(sick[len(sick) - 2].y)), 3)

#virusMain.py
def main():
    pygame.init()
    #screen setup
    pygame.display.set_caption("Simulasi Covid 19")
    screen.fill(pygame.Color("white"))

    #clock setup
    clock = pygame.time.Clock()
    MAX_FPS = 20

    #variables
    running = True
    numPeople = 300


    #create people
    patientZero1 = Person(random.randint(20, WIDTH/2 - 20), random.randint(20, HEIGHT/2-20), "sick", False)
    patientZero2 = Person(random.randint(WIDTH/2 + 20, WIDTH - 20),random.randint(20, HEIGHT/2 - 20), "sick", False)
    patientZero3 = Person(random.randint(20, WIDTH/2 - 20),random.randint(HEIGHT/2 + 20, HEIGHT - 20), "sick", False)
    patientZero4 = Person(random.randint(WIDTH/2 + 20, WIDTH - 20), random.randint(HEIGHT/2 +20, HEIGHT - 20), "sick", False)
    people1 = [patientZero1]
    people2 = [patientZero2]
    people3 = [patientZero3]
    people4 = [patientZero4]
    for i in range(80 - 1):
        people1.append(Person(random.randint(20, WIDTH/2 - 20), random.randint(20, HEIGHT/2-20), "healthy", False))
    for i in range(80 - 1):
        people2.append(Person(random.randint(WIDTH/2 + 20, WIDTH - 20),random.randint(20, HEIGHT/2 - 20), "healthy", False))
    for i in range(80 - 1):
        people3.append(Person(random.randint(20, WIDTH/2 - 20),random.randint(HEIGHT/2 + 20, HEIGHT - 20), "healthy", False))
    for i in range(80 - 1):
        people4.append(Person(random.randint(WIDTH/2 + 20, WIDTH - 20), random.randint(HEIGHT/2 +20, HEIGHT - 20), "healthy", False))

    while running:
        for event in pygame.event.get():
            if event.type == pygame.QUIT: #clicked "x" in top right
                running = False

        #update people
        for person in people1:
            x = person.update(screen, people1)
            if x == 2:
                people1.remove(person)
        for person in people2:
            x = person.update(screen, people2)
            if x == 2:
                people2.remove(person)
        for person in people3:
            x = person.update(screen, people3)
            if x == 2:
                people3.remove(person)
        for person in people4:
            x = person.update(screen, people4)
            if x == 2:
                people4.remove(person)

        #update graphics
        screen.fill(pygame.Color("white"))
        sick1 = []
        sick2 = []
        sick3 = []
        sick4 = []
        for person in people1:
            make_line(person,sick1)
        for person in people2:
            make_line(person,sick2)
        for person in people3:
            make_line(person,sick3)
        for person in people4:
            make_line(person,sick4)
        pygame.display.flip()

        #pause for frames
        clock.tick(MAX_FPS)

    pygame.quit()

main()