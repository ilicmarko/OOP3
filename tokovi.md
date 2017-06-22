# Sadržaj
- [Binarni tokovi](#binarni-tokovi)
	- [Nasleđuju:](#nasleuju)
	- [Klase:](#klase)
	- [Primer:](#primer)
- [Tokovi karaktera](#tokovi-karaktera)
	- [Nasleđuju:](#nasleuju)
	- [Klase:](#klase)
	- [Primer:](#primer)
- [Baferovani tokovi](#baferovani-tokovi)
	- [Klase:](#klase)
- [Data tokovi](#data-tokovi)
	- [Nasleđuju:](#nasleuju)
	- [Klase:](#klase)
- [Tokovi objekta](#tokovi-objekta)
	- [Nasleđuju:](#nasleuju)
	- [Klase:](#klase)

# Tokovi podataka

Paket `java.io` sadži skoro sve potrebne klase za ulaz i izlaz. Podržani su različiti
tipovi izlaza kao: primitivni, objekti, karakteri i drugo...

Postoje dva tipa tokova:
- **InputStream** - Ulazni tok, koristi se za čitanje.
- **OutputStream** - Izlazni tok, koristi se za pisanje.

## Binarni tokovi
Binarni tokovi se koriste za upisivanje i ispisivanje **8-bitnih bajtova**.

#### Nasleđuju:
- `InputStream`
- `OutputStream`

#### Klase:
- `FileInputStream`
- `FileOutputStream`

#### Primer:
Kopiranje fajla:

```java
try {
    in = new FileInputStream("input.jpg");
    out = new FileOutputStream("output.jpg");

    int c;
    while ((c = in.read()) != -1) {
        out.write(c);
    }
} finally {
    if (in != null) {
        in.close();
    }
    if (out != null) {
        out.close();
    }
}
```

Binarnim tokom je moguće iskopirati tekstualni fajl ali kako je I/O ograničen na
**8-bine bajtove**, to neće biti dobro za unicode karakter. Za to treba koristiti
**tokove karaktera**.

## Tokovi karaktera
Tokovi karaktera mogu da upisuju i ispisuju **16-bitni unicode**. Ovo omogućava da
upisujemo i ispisujemo fajlove koji sadrže više od ASCII karaktera.

#### Nasleđuju:
- `Reader`
- `Writer`

#### Klase:
- `FileReader`
- `FileWriter`

#### Primer:
Kopiranje fajla:

```java
try {
    in = new FileReader("input.txt");
    out = new FileWriter("output.txt");

    int c;
    while ((c = in.read()) != -1) {
        out.write(c);
    }
} finally {
    if (in != null) {
        in.close();
    }
    if (out != null) {
        out.close();
    }
}
```

## Baferovani tokovi
Prethodni primeri su bili *nebaferovani tokovi*. Svaki zahtev za čitanje i pisanje
je pozivao operacije diska. Što je činilo program neefikasnim i skupim.

Zato su uvedeni baferovani tokovi. Baferovani tokovi čitaju i pišu iz memorijskog
dela koji je poznatiji kao *bafer*. Kad se bafer napuni poziva se API za pražnjenje (ispis).

Vrlo lako možemo nebaferovane tokove prebaciti u baferovane, obmotavanjem.

```java
inputStream = new BufferedReader(new FileReader("example.txt"));
outputStream = new BufferedWriter(new FileWriter("out.txt"));
```

#### Klase:
- `BufferedReader` - Tok karaktera
- `BufferedReader` - Tok karaktera
- `BufferedInputStream` - Binarni tok
- `BufferedOutputStream` - Binarni tok

Bafer se obično prazni kada se napuni, ali je ponekad potrebno eksplicitno ga isprazniti.
To je poznatije kao *flushing* bafera.

Pozivanjem metode `flush()` tok će se ispisati.

## Data tokovi
Koriste se za ispisivanje primitivnih tipova (`boolean`, `char`, `byte`, `short`, `int`, `long`, `float`, i `double`)
takođe i String-ova.

Pri čitanju i upisu mora da se vodi računa o redosledu podataka.

#### Nasleđuju:
- `DataInput`
- `DataOutput`

#### Klase:
- `DataInputStream`
- `DataOutputStream`

## Tokovi objekta
Koriste se za serijalizaciju objekta i kasnije rekonstrukciju istog.
Metode `writeObject()` i `readObject()` se koriste za upisivanja i rekonstrukciju objekta.

`writeObject()` prolazi kroz objekat i u slučaju reference na neki drugi objekat, ućiće
u taj objekat i njega takođe serijalizovati. Tako da pozivanjem ove funkcije mogu da se
ispišu više objekta.

Objekat koji se serijalizuje mora da implementira `Serializable` interfejs.

![Vise objekta](http://i.imgur.com/UenKjlB.png)

#### Nasleđuju:
- `ObjectInput`
- `ObjectOutput`
`ObjectInput` i `ObjectOutput` su pod interfejsi `DataInput` i `DataOutput` interfejsima.

#### Klase:
- `ObjectInputStream`
- `ObjectOutputStream`
