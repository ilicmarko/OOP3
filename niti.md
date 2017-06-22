# Procesi i niti
Nit je jedinica posla, dok je jedan proces skup jedne ili više niti koji dobijaju
svoju grupu resursa **(def. by Krca!)**. Niti ne mogu da se nađu van procesa i više
niti u jednom procesu dele resurse tog procesa.

Pravljenje nove niti zahreva mnogo manje resursa nego pravljenje nove niti.

Svaka nit je povezana sa klasom `Thread`. Postoje dve strategije za pravljenje niti:
- **Direktno** - Napravimo novu instancu klase `Thread` kada je potrebno pokrenuti asinhrono izvršenje.
- **Apstraktno** - Prosledimo zahtev za izvršavanje `executor`-u.

## Stanja niti
Tokom životnog veka nit prođe kroz različita stanja.
- **New** - Nit započinje svoj život u ovom stanju i nalazi se u njemu sve dok program ne započne nit.
- **Runnable** - Kad se nad novo nastalom niti pozove `start()`, nit postaje `runnable`. Čime se smatra da se izvršava.
- **Waiting** - Nit se nalazi u ovom stanju kada čeka drugi proces ili čeka određeni vremenski period.
- **Dead** - Nit ulazi u ovo stanje ako je završio posao ili je abnormalno prekinut.

![Thread zivot](http://i.imgur.com/OxHhgwo.png)

## Pravljenje niti
Aplikacija koja pravi instancu `Thread` mora da obezbedi kod koji će da pokrene tu nit.

### Runnable objekat
`Runnable` interfejs pruža jednu jedinu metodu `run`, koja sadrži kod za izvršavanje unutar niti.

```java
public class HelloRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Hello from a thread!");
    }
    public static void main(String args[]) {
        (new Thread(new HelloRunnable())).start();
    }
}
```

### Thread objekat
Drugi način je nasleđivanje klase `Thread` (ona svakako u sebi nasleđuje `Runnable` interfejs).

```java
public class HelloThread extends Thread {
    @Override
    public void run() {
        System.out.println("Hello from a thread!");
    }
    public static void main(String args[]) {
        (new HelloThread()).start();
    }
}
```

## Pauziranje izvršenja
`Thread.sleep` zaustavlja izvršavanje trenutne niti. Pauzu je moguće prekinuti `interrupt`-om.
Postoje još dva načina pozivanje funkcije, gde je parametar vreme čekanja u nanosekundama ili sekundama.

*Vreme čekanje ne mora da bude tačno zato što zavisi od operativnog sistema.*

```java
public class SleepMessages {
    public static void main(String args[]) throws InterruptedException {
        String importantInfo[] = {
            "Mares eat oats",
            "A kid will eat ivy too" };
        for (int i = 0; i < importantInfo.length; i++) {
            Thread.sleep(4000);
            System.out.println(importantInfo[i]);
        }
    }
}
```
#### Izuzeci
- InterruptedException

## Prekidanje niti
`interrupt` služi kao indigator (flag) kad nit treba da prestane sa onim što radi i radi nešto drugo.
Kako će se obraditi prekid to zavisi od programa.

### Prekidanje
Način na koji se prekid implementira jeste preko izuzetaka, tako što se nad metodima koji bacaju
`InterruptedException` izuzetak pozove taj izuzetak i prekid se obradi u `catch` bloku.

```java
for (int i = 0; i < importantInfo.length; i++) {
    try {
        Thread.sleep(4000);
    } catch (InterruptedException e) {
        // Ovde se nas prekid obradi
        return;
    }
    System.out.println(importantInfo[i]);
}
```

Drugi način je ako imamo neki veliki proces koji se izvršava jeste, da konstantno proveravamo
da li je nit prekinut. To radimo pozivom `Thread.interrupted()`.

```java
for (int i = 0; i < inputs.length; i++) {
    heavyCrunch(inputs[i]);
    if (Thread.interrupted()) {
        // Doslo je do prekida uradi nesto
        return;
    }
}
```

### Indikator
Mehanizam za prekide je implementiran preko flag-a poznatiji kao `interrupt status`.
Pozivom metodom `Thread.interrupt` postavlja se flag. Indikator se proverava na dva načina:

- `Thread.interrupted()` - **Statička** metod koja pri proveri briše flag.
- `isInterrupted()` - **Ne statička** metoda koja pri proveri ostavi flag.

Po konvenciji, metod koji baci `InterruptedException` briše indikator.

## Čekanje na završetak
`join` metoda omogućava da jedna nit sačeka izvršenje druge.

Ako je `t` promenjiva koja je `Thread` objekat, i trenutno se izvršava.
Pozivanjem `t.join()` ima kao posledicu da se trenutna nit zaustavlja sve dok se
`t` ne završi sa izvršavanjem.

Takođe kao kod metode `sleep` možemo proslediti vreme čekanja.

#### Izuzeci
- InterruptedException


## Krajnji primer
```java
public class SimpleThreads {
    // Display a message, preceded by
    // the name of the current thread
    static void threadMessage(String message) {
        String threadName =
            Thread.currentThread().getName();
        System.out.format("%s: %s%n", threadName, message);
    }

    private static class MessageLoop
        implements Runnable {
        public void run() {
            String importantInfo[] = {
                "Mares eat oats",
                "Does eat oats",
                "Little lambs eat ivy",
                "A kid will eat ivy too"
            };
            try {
                for (int i = 0;
                     i < importantInfo.length;
                     i++) {
                    // Pause for 4 seconds
                    Thread.sleep(4000);
                    // Print a message
                    threadMessage(importantInfo[i]);
                }
            } catch (InterruptedException e) {
                threadMessage("I wasn't done!");
            }
        }
    }

    public static void main(String args[])
        throws InterruptedException {

        // Delay, in milliseconds before
        // we interrupt MessageLoop
        // thread (default one hour).
        long patience = 1000 * 60 * 60;

        // If command line argument
        // present, gives patience
        // in seconds.
        if (args.length > 0) {
            try {
                patience = Long.parseLong(args[0]) * 1000;
            } catch (NumberFormatException e) {
                System.err.println("Argument must be an integer.");
                System.exit(1);
            }
        }

        threadMessage("Starting MessageLoop thread");
        long startTime = System.currentTimeMillis();
        Thread t = new Thread(new MessageLoop());
        t.start();

        threadMessage("Waiting for MessageLoop thread to finish");
        // loop until MessageLoop
        // thread exits
        while (t.isAlive()) {
            threadMessage("Still waiting...");
            // Wait maximum of 1 second
            // for MessageLoop thread
            // to finish.
            t.join(1000);
            if (((System.currentTimeMillis() - startTime) > patience)
                  && t.isAlive()) {
                threadMessage("Tired of waiting!");
                t.interrupt();
                // Shouldn't be long now
                // -- wait indefinitely
                t.join();
            }
        }
        threadMessage("Finally!");
    }
}
```
