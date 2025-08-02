# Vylepšená integrace Zásilkovny pro SimpleShop

Pokročilá integrace výběru pobočky Zásilkovny do SimpleShop formuláře s optimalizovaným mobilním zobrazením. Řeší problémy původní integrace a přidává podporu pro responzivní design.

*Vychází z původního projektu: https://github.com/MiliusCZ/simpleshop-zasilkovna*

![Ukázka](https://github.com/mediatoring/simpleshop-zasilkovna/blob/main/ukazka.png?raw=true)

## Požadavky

- **SimpleShop účet**: https://www.simpleshop.cz/?utm=dwso3bv
- **Packeta/Zásilkovna účet**: https://www.packeta.com

## Nastavení v SimpleShop

### 1. Konfigurace dopravních variant

Vytvořte v produktu položky doplňkového prodeje reprezentující různé způsoby dopravy. Jedna z nich bude představovat doručení přes Zásilkovnu.

![Nastavení doplňkového prodeje](https://github.com/mediatoring/simpleshop-zasilkovna/blob/main/doplnkovy%20prodej.png?raw=true)

### 2. Přidání formulářového pole

V sekci *Formulář* při úpravě produktu vytvořte nové pole typu *Jednořádkový text* s názvem **Adresa zásilkovny**. **Důležité**: Označte pole jako povinné, aby zákazníci nemohli odeslat objednávku bez výběru pobočky.

![Vlastní pole ve formuláři](https://github.com/mediatoring/simpleshop-zasilkovna/blob/main/vlastn%C3%AD%20pole.png?raw=true)

### 3. Implementace kódu

Vložte následující kód do záložky *Ostatní* → *JS, CSS a jiné kódy* v nastavení produktu.

**Před použitím upravte tyto parametry** (hodnoty najdete v HTML kódu formuláře pomocí *Prozkoumat prvek*):

- `inputName` - name atribut textového pole "Adresa zásilkovny"
- `hiddenFieldName` - name atribut skrytého pole "Adresa zásilkovny"  
- `transportSelectorName` - name atribut pro výběr dopravy
- `zasilkovnaValue` - index Zásilkovny v seznamu doprav (začíná od 0)
- `YOURAPIKEY` - váš API klíč z Packeta portálu

```html
<style>
/* Responzivní styly pro mobilní zařízení */
@media (max-width: 768px) {
    #packeta-widget-frame {
        width: 100% !important;
        height: 100% !important;
        position: fixed !important;
        top: 0 !important;
        left: 0 !important;
        bottom: 0 !important;
        right: 0 !important;
        z-index: 9999 !important;
        border-radius: 0 !important;
    }
    
    #packeta-widget-frame iframe {
        width: 100% !important;
        height: 100% !important;
    }
    
    .packeta-widget-search {
        font-size: 16px !important;
        padding: 10px !important;
        margin: 10px !important;
    }
    
    .packeta-widget-button {
        font-size: 16px !important;
        padding: 12px 20px !important;
        margin: 5px !important;
    }
}

/* Styling tlačítka pro výběr pobočky */
.packeta-selector-open {
    background-color: #39b54a;
    color: white;
    border: none;
    padding: 10px 15px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    transition: background-color 0.3s ease;
}

.packeta-selector-open:hover {
    background-color: #2d8f3a;
}
</style>

<script type="text/javascript">
    /* === KONFIGURACE - UPRAVTE PODLE VAŠEHO FORMULÁŘE === */
    var inputName = 'payment[-::jIzui4tg::value]';
    var hiddenFieldName = 'payment[-::jIzui4tg::name]';
    var transportSelectorName = 'payment[items::1::key_radio]';
    var zasilkovnaValue = '2';
    /* === KONEC KONFIGURACE === */

    function initZasilkovna() {
        var zasilkovnaInput = document.querySelector("input[name='" + inputName + "']");
        var zasilkovnaHidden = document.querySelector("input[name='" + hiddenFieldName + "']");

        // Čekání na načtení formuláře
        if (!zasilkovnaInput || !zasilkovnaHidden) {
            setTimeout(initZasilkovna, 500);
            return;
        }

        var zasilkovnaParent = zasilkovnaInput.parentNode.parentNode;

        // Vytvoření tlačítka pro výběr pobočky
        var selectZasilkovnaButton = document.createElement('button');
        selectZasilkovnaButton.textContent = 'Vybrat pobočku';
        selectZasilkovnaButton.setAttribute('class', 'packeta-selector-open');
        selectZasilkovnaButton.setAttribute('type', 'button');

        // Přidání tlačítka do formuláře
        zasilkovnaInput.parentNode.insertBefore(selectZasilkovnaButton, zasilkovnaInput);
        zasilkovnaInput.setAttribute('class', 'packeta-selector-branch-name');
        zasilkovnaHidden.setAttribute('class', 'packeta-selector-branch-id');
        zasilkovnaInput.setAttribute('readonly', 'readonly');
        zasilkovnaInput.parentNode.style.display = 'flex';

        // Logika zobrazení/skrytí pole podle výběru dopravy
        var showHideZasilkovna = function(value) {
            var state = value === zasilkovnaValue ? 'block' : 'none';
            zasilkovnaParent.style.display = state;
            zasilkovnaInput.value = state === 'block' ? '' : 'N/A';
        }

        // Přidání event listenerů na výběr dopravy
        var transportSelectors = document.querySelectorAll("input[name='" + transportSelectorName + "']");
        for (var i = 0; i < transportSelectors.length; i++) {
            transportSelectors[i].addEventListener('change', function() {
                showHideZasilkovna(this.value);
            });
            if ((transportSelectors[i].value === zasilkovnaValue) && !transportSelectors[i].checked) {
                zasilkovnaParent.style.display = 'none';
            }
        }

        // Inicializace Packeta widgetu
        setTimeout(initPacketaWidget, 500);
    }

    // Spuštění po načtení stránky
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', function() {
            setTimeout(initZasilkovna, 1000);
        });
    } else {
        setTimeout(initZasilkovna, 1000);
    }
</script>

<script>
    // Konfigurace Packeta Widget v6
    var packetaOptions = {
        language: 'cs',
        country: 'cz',
                vendors: [
            {
                country: 'cz',
                group: ''
            },
            // Přidali jsme 'zbox' do seznamu povolených skupin, aby se zobrazily i výdejní boxy.
            // Pokud Zboxy nechcete, odstraňte následující 4 řádky od { po } a současně raději i čárku za }, o dva řádky výš
            {
                country: 'cz',
                group: 'zbox'
            }
        ]
    };
    
    function initPacketaWidget() {
        var buttons = document.querySelectorAll('.packeta-selector-open');
        var branchNameInput = document.querySelector('.packeta-selector-branch-name');
        var branchIdInput = document.querySelector('.packeta-selector-branch-id');
        
        if (!buttons.length || !branchNameInput || !branchIdInput) {
            setTimeout(initPacketaWidget, 500);
            return;
        }
        
        buttons.forEach(function(button) {
            button.addEventListener('click', function() {
                if (typeof Packeta !== 'undefined' && Packeta.Widget) {
                    Packeta.Widget.pick('YOURAPIKEY', function(point) {
                        if (point) {
                            branchNameInput.value = point.name;
                            branchIdInput.value = point.id;
                        }
                    }, packetaOptions);
                }
            });
        });
    }
</script>

<script src="https://widget.packeta.com/v6/www/js/library.js"></script>
```

## Klíčová vylepšení

✅ **Mobilní optimalizace** - Widget se správně zobrazí na všech mobilních zařízeních  
✅ **Asynchronní načítání** - Čeká na kompletní načtení SimpleShop formuláře  
✅ **Moderní API** - Využívá nejnovější Packeta Widget v6 s lepší stabilitou  
✅ **Responzivní design** - Automatické přizpůsobení velikosti obrazovky  
✅ **Vylepšené UX** - Intuitivní ovládání s hover efekty

## Implementace

1. **Zkopírujte a upravte** konfigurační parametry podle vašeho formuláře
2. **Vložte kód** do SimpleShop produktu v sekci *Ostatní* → *JS, CSS a jiné kódy*
3. **Otestujte funkčnost** na desktopu i mobilu
4. **Opakujte proces** pro každý produkt s doručením přes Zásilkovnu

## Podpora a řešení problémů

**Časté problémy:**
- ❌ **Widget se neotevře**: Zkontrolujte platnost API klíče
- ❌ **Nesprávné parametry**: Ověřte konfigurační hodnoty v HTML kódu
- ❌ **Mobilní zobrazení**: Zkontrolujte CSS styly pro responzivní design

**Debugging:**
- Otevřete vývojářské nástroje (F12) → Console
- Zkontrolujte chybové hlášky
- Ověřte, že je vybrána doprava "Zásilkovna"

---

**Technická podpora**: https://www.mediatoring.cz

*Poznámka: Integrace umožňuje pouze výběr pobočky. Vytvoření zásilky v systému Zásilkovny musíte provést ručně nebo pomocí CSV importu.*

### Podmínky použití

- ✅ **Komerční použití povoleno** - Můžete kód používat v komerčních projektech
- ✅ **Modifikace povolena** - Smíte kód upravovat podle svých potřeb
- ✅ **Distribuce povolena** - Můžete kód sdílet s ostatními
- ✅ **Soukromé použití** - Volné použití pro osobní projekty

**Povinnosti:**
- 📝 Zachovat copyright notice při distribuci
- 📝 Přiložit kopii licence při šíření

**Omezení:**
- ❌ Žádná záruka funkčnosti
- ❌ Autoři nenesou odpovědnost za škody

### Uznání autorství

Pokud tento kód používáte ve svém projektu, budeme rádi za uvedení odkazu na původní repozitář nebo na **www.mediatoring.cz**.
