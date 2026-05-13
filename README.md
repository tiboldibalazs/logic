def ures_sor_e(sor: str) -> bool:
    """
    Megnézi, hogy üres-e a beolvasott sor.
    
    >>> ures_sor_e("  ")
    True
    >>> ures_sor_e("anna;Dune;14")
    False
    """
    return sor.strip() == ""

def vege_sor_e(sor: str) -> bool:
    """
    Ellenőrzi, hogy a sor a lezáró VEGE szó-e.
    
    >>> vege_sor_e("VEGE")
    True
    """
    return sor.strip() == "VEGE"

def kolcsonzes_sor_feldolgozasa(sor: str) -> tuple[str, str, int]:
    """
    Szétvágja a sort és visszaadja az adatokat.
    
    >>> kolcsonzes_sor_feldolgozasa("anna;Dune;14")
    ('anna', 'Dune', 14)
    >>> kolcsonzes_sor_feldolgozasa("hibas sor")
    Traceback (most recent call last):
    ...
    ValueError: Hibásan formázott kölcsönzési sor
    """
    reszek = sor.split(";")
    if len(reszek) != 3:
        raise ValueError("Hibásan formázott kölcsönzési sor")
    
    nev = reszek[0].strip()
    konyv = reszek[1].strip()
    
    if nev == "" or konyv == "":
        raise ValueError("Hibásan formázott kölcsönzési sor")
        
    try:
        napok = int(reszek[2].strip())
        if napok < 0:
            raise ValueError("Hibásan formázott kölcsönzési sor")
        return (nev, konyv, napok)
    except ValueError:
        raise ValueError("Hibásan formázott kölcsönzési sor")

def olvasok_sora_feldolgozasa(sor: str) -> tuple[str, str]:
    """
    Kinyeri a két összehasonlítandó nevet.
    
    >>> olvasok_sora_feldolgozasa("anna bela")
    ('anna', 'bela')
    """
    nevek = sor.split()
    return (nevek[0], nevek[1])

def naplo_feldolgozasa(sorok: list[str]) -> dict[str, dict[str, list[int]]]:
    """
    Felépíti a napló szótárat a sorokból.
    """
    adatok = {}
    for sor in sorok:
        if ures_sor_e(sor) or vege_sor_e(sor):
            continue
        try:
            nev, konyv, nap = kolcsonzes_sor_feldolgozasa(sor)
            if nev not in adatok:
                adatok[nev] = {}
            if konyv not in adatok[nev]:
                adatok[nev][konyv] = []
            adatok[nev][konyv].append(nap)
        except ValueError:
            continue
    return adatok

def olvaso_napjai(tarolo: dict, olvaso: str) -> list[int]:
    """
    Összegyűjti egy olvasó összes napját.
    
    >>> t = {'a': {'K1': [5, 10]}}
    >>> olvaso_napjai(t, 'a')
    [5, 10]
    """
    if olvaso not in tarolo:
        return []
    lista = []
    for konyv_napjai in tarolo[olvaso].values():
        for n in konyv_napjai:
            lista.append(n)
    return lista

def olvaso_konyvei(tarolo: dict, olvaso: str) -> set[str]:
    """
    Visszaadja az olvasó által olvasott egyedi könyveket.
    """
    if olvaso not in tarolo:
        return set()
    return set(tarolo[olvaso].keys())

def atlag(napok: list[int]) -> float:
    """
    Kiszámolja az átlagot.
    
    >>> atlag([10, 20])
    15.0
    """
    if len(napok) == 0:
        return 0.0
    return sum(napok) / len(napok)

def statisztika_keszitese(tarolo: dict, o1: str, o2: str) -> dict[str, object]:
    """
    Létrehozza a statisztikai eredményeket.
    """
    n1 = olvaso_napjai(tarolo, o1)
    n2 = olvaso_napjai(tarolo, o2)
    k1 = olvaso_konyvei(tarolo, o1)
    k2 = olvaso_konyvei(tarolo, o2)
    
    ossz1 = sum(n1)
    ossz2 = sum(n2)
    
    jobb = "dontetlen"
    if ossz1 > ossz2:
        jobb = o1
    elif ossz2 > ossz1:
        jobb = o2
        
    return {
        "o1_ossznap": ossz1,
        "o2_ossznap": ossz2,
        "o1_konyv_db": len(k1),
        "o2_konyv_db": len(k2),
        "kozos_konyv_db": len(k1 & k2),
        "osszes_konyv_db": len(k1 | k2),
        "o1_atlag": round(atlag(n1), 2),
        "o2_atlag": round(atlag(n2), 2),
        "tobbet_kolcsonzott": jobb
    }

def statisztika_szovegge(statisztika: dict) -> str:
    """Kiírható szöveggé formázza az adatokat."""
    szoveg = ""
    for kulcs in statisztika:
        szoveg += kulcs + "=" + str(statisztika[kulcs]) + "\n"
    return szoveg.strip()
