from pyo import *
import random
s = Server().boot()
s.start()

gamme=[0,2,3,5,7,9,10]

timeMul=[0.5, 1.5, 1, 1, 0.25, 0.33,2]


meltemps=0.25
bassetemps=meltemps*4
padtemps=meltemps*8


class Melodie: 
    def __init__ (self):
        
        self.env=Adsr(attack=0.01, decay=0.05, sustain=0.71, release=0.10, dur=meltemps, mul=1.2)
        self.src=LFO(freq=400, sharp=0.5, type=3, mul=self.env*.5)
        self.reverb=WGVerb(self.src, feedback=0.50, cutoff=5000, bal=0, mul=1, add=0)
        self.filtre=MoogLP(self.reverb, freq=33)
        self.delay = Delay(self.filtre, mul=0)
        self.forout=self.filtre+self.delay
        self.out = self.forout.mix(2).out()
        
        
    def play(self):
        newTime=random.choice(timeMul)
        self.env.dur=newTime
        patMelodie.time=newTime
        self.src.freq=midiToHz(random.choice(gamme)+60)


        self.env.play()
        
    def change(self, filtreres = 0, filtFreq = 0, delFeed=0.5, delmul=0, balrev=0):
        self.filtre.set("res",filtreres,10)
        self.filtre.set("freq",filtFreq,10)
        self.delay.feedback = delFeed
        self.delay.set("mul", delmul, 10)
        self.reverb.set("bal", balrev, 8)

class Basse:
    def __init__ (self):
        
        self.env=Adsr(attack=0.01, decay=0.05, sustain=0.71, release=0.10, dur=bassetemps, mul=0)
        self.a=SuperSaw(freq=100, detune=0.50, bal=0.70, mul=self.env*.2, add=0)
        self.out=self.a.mix(2).out()
        
    def play(self):
        
    
        
        self.a.freq=midiToHz(random.choice(gamme)+24)


        self.env.play()
        
    def change(self, envMul=0):
        self.env.set("mul", envMul, 10)

class Pad:
    def __init__ (self):
        
        
        self.env=Adsr(attack=0.01, decay=0.05, sustain=0.71, release=0.10, dur=bassetemps, mul=1)
        self.a=[LFO(freq=400, sharp=random.random(), type=random.choice([0,1,2,3,7]), mul=.1) for i in range(10)]
        self.b=sum(self.a).mix(2)
        self.filtre=MoogLP(self.b, freq=200)
        self.delay = Delay(self.filtre, mul=0)
        self.forout=self.filtre+self.delay
        self.out = self.forout.out()


    def play(self):
        
        for i in range(10):
        
            self.a[i].freq=midiToHz(random.choice(gamme)+48)


    def change(self, filtreres = 0, filtFreq = 0, delFeed=0.5, delmul=0):
        self.filtre.set("res",filtreres,10)
        self.filtre.set("freq",filtFreq,10)
        self.delay.feedback = delFeed
        self.delay.set("mul", delmul, 10)
        
        


mel=Melodie()
bass=Basse()
pad=Pad()

patMelodie=Pattern(mel.play, time=meltemps).play()
patBasse=Pattern(bass.play, time=bassetemps).play()
patPad=Pattern(pad.play,time=padtemps).play()


#pad.def change(self, filtreres = 0, filtFreq = 0, delFeed=0.5, delmul=0) 
#bass.def change(self, envMul=0)
#Mel.def change(self, filtreres = 0, filtFreq = 0, delFeed=0.5, delmul=0, balrev=0)


def event_0():
    
    pad.change(1, 1000) 
    bass.change(1)
    mel.change(1, 1000)

    
def event_1():
    pad.change(0.8, 1500, 0.8) 
    bass.change(1)
    mel.change(0.7, 800, 0.7, 0.4)
    
        
def event_2():
    pad.change(random.uniform(0.2,0.6), 1500, 0.8) 
    bass.change(1)
    mel.change(0.9, 600, 0.7, 0.3, 0.8)
    
def event_3():
    global gamme
    gamme=[x+3 for x in [0,2,3,5,7,9,10,12]]
    pad.change(random.uniform(0.6,0.9), 800, 0.5) 
    bass.change(1)
    mel.change(0.7, 900, 0.8, 0.5, 0.9)
    
def event_4():
    global gamme
    gamme=[0,2,3,5,7,9,10,12]
    pad.change()
    bass.change()
    mel.change()
    metro.time=10
    
def event_5():
    metro.stop()
    s.stop()



metro=Metro(20).play()

c = Counter(metro, min=0, max=6)

score=Score(c, fname="event_")



  
s.gui(locals())








