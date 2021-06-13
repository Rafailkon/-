## Διαδραστικός Ηχητικός Σχεδιασμός (ADA135)
- `Eργασία Εξαμήνου`

### Ραφαήλ Κωνσταντινίδης `(ΑΜ: ADA2020011)` 

H παρούσα εργασία αφορά την πρώτη μου πειραματική απόπειρα υλοποίησης ενός ηχοτοπίου στο προγραμματιστικό περιβάλλον του Supercollider.
Πρωτεύων χαρακτηριστικό της σύνθεσης αποτελεί το στοιχείο του τυχαίου. Χρησιμοποιώντας τις διάφορες εντολές και εκμεταλλευόμενος τις δυνατότητες 
των τυχαίων παραμέτρων που μπορούν να τεθούν στον κώδικα, τόσο στις γεννήτριες `LFNoise`, όσο και στην ακολουθία `Pwhite , Pexprand, Prand`,
προσπάθησα να προσδώσω την δυνατότητα της αυτονομίας και αυτοοργάνωσης του ηχητικών στοιχείων. Kατά αυτό τον τρόπο αποφεύχθηκε η αυτούσια 
επανάληψη της ακολουθίας `Pbind`, δημιουργώντας παράλληλα μια πιο προοδευτική σύνθεση. Παρόλο που άφησα το μέσο να με καθοδηγήσει στο
τελικό αποτέλεσμα, προσδιόρισα το εύρος των τιμών που θα κυμαίνονταν οι ήχοι. Χρησιμοποίησα τρεις γεννήτριες ήχου `Pulse.ar..SinOsc.ar`,
η μια εκ των οποίων είναι θόρυβος `PinkNoise.ar`. Με τις ανάλογες περιβάλλουσες `EnvGen.kr` και τη χρήση φίλτρων `BPF.ar ... RLPF.ar`
θέλησα να προσμοιάσω οργανικούς και φυσικούς ήχους, καθώς και την ανάδειξη τους μέσω του χώρου. Για το σκοπό αυτό δημιούργησα Reverb `SynthDef(\reverb..`
καθώς και στερεοφωνία με την χρήση του `Pan2.ar`.

Mετά την υλοποίηση του κώδικα αποφάσισα να ηχοφραφήσω ένα ηχητικό απόσπαμα `s.record;`, σαν απόδοση αυτής της προσπάθειας. Εφόσον η ηχογράφηση υφίσταντο 
σε συγκεκριμένο χρόνο, αντιμετώπισα αυτό το αποτέλεσμα ως μια σκηνή. Αυτή η  αποτύπωση - καταγραφή της χρονικής αυτής στιγμής με ώθησε στην μετέπειτα οπτικοποίηση της. 
Για το σκοπό αυτό χρησιμοποίησα το Processing με την εισαγωγή της βιβλιοθήκης `processing.sound` και το Fast Fourier transform `FFT fft;`, δημιουργώντας το 
φασματογραφήμα (spectogram) του ηχητικού αποσπάσματος. Σαν εννοιολογικό πλαίσο της εργασίας εντάσσονται οι έννοιες του χρόνου, της φθορά και της μνήμης, καθώς
εμπίτπτουν και απορρέουν στην διαδικασία της αποτύπωσης.

## [Video](https://1drv.ms/v/s!AjVIyz1h0tNBhkb32xqncUZg7rm3?e=GvXYze)

<details>
  <summary>Super Collider code</summary>

``` sclang
s.meter;
s.plotTree;
(
SynthDef(\synthP, {
	arg atk =2 , sus =0, rel=5, c1=1, c2=(-1),
	freq = 500, cfmin = 500, cfmax=2000, rqmin = 0.1,rqmax = 0.2, amp =1, detune=0.2,pan=0, out=0;
	var sig, env;
	env = EnvGen.kr(Env([0,1,1,0],[atk, sus, rel],[c1,0,c2]),doneAction:2);
	sig = Pulse.ar(freq* {LFNoise1.kr(0.5,detune).midiratio});
	sig = BPF.ar(sig,{LFNoise1.kr(0.2).exprange(cfmin,cfmax)},
		             {LFNoise1.kr(0.1).exprange(rqmin,rqmax)});
        sig = Pan2.ar(sig, pan);
	sig = sig * env * amp;
	Out.ar(out, sig);
}).add;

SynthDef(\pad,{
	arg freq = 440, atk= 0.005, rel = 0.3, amp = 1, pan=0, out = 0;
	var sig, env;
	sig = SinOsc.ar([freq,freq-223,4]);
	env = EnvGen.kr(Env.new([0,1,0],[atk,rel],[1,-1]),doneAction:2);
	sig = Pan2.ar(sig, pan, amp);
	sig = sig * env;
	Out.ar(out, sig);
}).add;

SynthDef(\noise,{
	arg out = 0 ;
        var sig;
	sig = PinkNoise.ar(0.5!2);
	2.do{sig = RLPF.ar(sig,LFNoise2.kr(0.5).exprange(100,5000))};
    Out.ar(out,sig);
}).add;

SynthDef(\reverb,{
	arg in, predelay = 0.1, revtime = 3, lpf = 4500, mix = 1, amp=1, out=0;
	var dry, wet, temp, sig;
	dry = In.ar(in,2);
	temp = In.ar(in,2);
	wet = 0;
	temp = DelayN.ar(temp, 0, 2, predelay);
	16.do{
		temp = AllpassN.ar(temp, 0.05, {Rand(0.001,0.05)}!2, revtime);
		temp = LPF.ar(temp, lpf);
		wet = wet + temp;
	};
	sig = XFade2.ar(dry,wet,mix*2-1,amp);
	Out.ar(out,sig);
}).add;
)
~reverbBus = Bus.audio(s,2);
~reverbSynth = Synth(\reverb,[\in,~reverbBus]);

(
~ambient = Pbind(
    \instrument, \pad,
    \dur, Pwhite(0.05, 0.5, inf),
	\midinote, 33,
	\harmonic, Pexprand(1, 80, inf).round,
	\atk, Pwhite(2.0, 3.0, inf),
	\rel, Pwhite(5.0, 3.0, inf),
	\atk, Pwhite(2.0, 3.0, inf),
	\amp, Pkey(\harmonic)*0.5,
	\pan, Pwhite(-0.8, 0.8, inf),
	\out, ~reverbBus
).play;
)

~ambient.stop;

~wind = Synth(\noise,[\out, ~reverbBus]);

~wind.free;

(
~drum =  Pbind(
	\instrument, \synthP,
	\dur,Prand([1,0.5],inf),
	\freq,Prand([0.4,8],inf),
	\detune,Pwhite(0,0.1,inf),
	\rqmin,0.005,
	\rqmax,0.008,
	\cfmin,Prand((Scale.minor.degrees+64).midicps,inf)*Prand([0.5,1,2,4],inf),
	\cfmax,Pkey(\cfmin)*Pwhite(1.008,1.025,inf),
	\atk, 3,
	\sus,1,
	\rel,5,
	\amp,7,
	\pan, Pwhite(-0.8, 0.8, inf),
	\out, ~reverbBus
).play;
)
~drum.stop;

s.record;
s.stopRecording;
 ``` 
  </details>
 
 <details>
  <summary>Processing code</summary>

```java 
import processing.sound.*;
SoundFile track;
FFT fft;       
int bands = 8192;
float[] spectrum = new float[bands]; 
float angle= -QUARTER_PI*3;

void setup()
{
  surface.setLocation(200,height/5);
  size(1000, 1000);
  background(0);
  fft = new FFT(this, bands);
  track = new SoundFile(this, "SuperC.aiff");
  track.loop();
  fft.input(track);
}
void draw() {
  fft.analyze();
  translate(width/2, height/2);
  rotate(angle);
  for (int i = 0; i<bands; i++)
    {
    strokeWeight(3);
    spectrum[i] = fft.spectrum[i]*25;
    stroke(255,spectrum[i]*255);  
    point(i,i);
    }
  angle += PI/(256*4)*0.1605; 
   
  
  if( angle >= QUARTER_PI*5){
    //background(255);
    //angle = -QUARTER_PI*3;
    noLoop();
    track.stop();
  }
}
  ```
  </details>
