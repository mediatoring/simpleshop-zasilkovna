# simpleshop-zasilkovna

Integrace výběru pobočky Zásilkovny do Simpleshop formuláře. Tato integrace přidá do formuláře pole s možností výběru výdejny zásilkovny s vylepšeným mobilním zobrazením. Postaveno na původním kódu https://github.com/MiliusCZ/simpleshop-zasilkovna

![Ukázka](https://github.com/MiliusCZ/simpleshop-zasilkovna/blob/main/ukazka.png?raw=true)

## Co je potřeba

- Účet pro Simpleshop (https://www.simpleshop.cz/)
- Účet pro Zásilkovnu (https://www.zasilkovna.cz/)

## Konfigurace v Simpleshopu

### Vytvoření variant dopravy

K produktu vytvořte položky doplňkového prodeje pro dopravu. Jedna z položek bude představovat dodání přes Zásilkovnu. 

![Ukázka nastavení doplňkového prodeje](https://github.com/MiliusCZ/simpleshop-zasilkovna/blob/main/doplnkovy%20prodej.png?raw=true)

### Vytvoření políčka ve formuláři

V záložce *Formulář* v úpravě produktu vytvořte nové pole typu *Jednořádkový text*. Název je libovolný - například **Adresa zásilkovny**. Pole je potřeba označit jako povinné, aby zákazníci nemohli odeslat formulář bez vybrané výdejny.

![Vlastní pole formuláře](https://github.com/MiliusCZ/simpleshop-zasilkovna/blob/main/vlastn%C3%AD%20pole.png?raw=true)

### Kód

Kód integrace vložte v úpravě produktu do záložky *Ostatní*, *pole JS, CSS a jiné kódy*. Před vložením je třeba v kódu upravit několik položek.
Hodnoty je třeba vyčíst ze zdrojového kódu stránky s formulářem (například pomocí pravého kliku na pole a vybrání položky *prozkoumat*). Hodnoty hledejte v náhledu prodejního formuláře, nikoliv v editaci.

- *inputName* - název input elementu pole **Adresa zásilkovny** 
- *hiddenFieldName* - název hidden input elementu pole **Adresa zásilkovny** 
- *transportSelectorName* - název atributu pro výběr doplňkového prodeje 
- *zasilkovnaValue* - pořadí zásilkovny v seznamu doplňkového prodeje. Počítá se od nuly, takže například bude-li Zásilkovna na druhém místě, hodnota je "1"

Nahraďte řetězec YOURAPIKEY v posledním řádku skriptu Vaším API klíčem, který najdete na portálu Zásilkovny

```html
<style>
/* CSS styly pro responsivní widget Zásilkovny */
@media (max-width: 768px) {
    /* Úprava kontejneru widgetu */
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
    
    /* Úprava mapy v mobilu */
    #packeta-widget-frame iframe {
        width: 100% !important;
        height: 100% !important;
    }
    
    /* Zajištění správného zobrazení vyhledávacího pole */
    .packeta-widget-search {
        font-size: 16px !important;
        padding: 10px !important;
        margin: 10px !important;
    }
    
    /* Úprava tlačítek */
    .packeta-widget-button {
        font-size: 16px !important;
        padding: 12px 20px !important;
        margin: 5px !important;
    }
}

/* Obecné úpravy pro lepší zobrazení */
.packeta-selector-open {
    background-color: #39b54a;
    color: white;
    border: none;
    padding: 10px 15px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
}

.packeta-selector-open:hover {
    background-color: #2d8f3a;
}
</style>

<script type="text/javascript">
    /* CONFIG PROPERTIES BEGIN */
    var inputName = 'payment[-::jIzui4tg::value]';
    var hiddenFieldName = 'payment[-::jIzui4tg::name]';
    var transportSelectorName = 'payment[items::1::key_radio]';
    var zasilkovnaValue = '2';
    /* CONFIG PROPERTIES END */

    // Funkce pro inicializaci Zásilkovny
    function initZasilkovna() {
        var zasilkovnaInput = document.querySelector("input[name='" + inputName + "']");
        var zasilkovnaHidden = document.querySelector("input[name='" + hiddenFieldName + "']");

        // Pokud prvky ještě neexistují, zkus to znovu za chvíli
        if (!zasilkovnaInput || !zasilkovnaHidden) {
            setTimeout(initZasilkovna, 500);
            return;
        }

        var zasilkovnaParent = zasilkovnaInput.parentNode.parentNode;

        var selectZasilkovnaButton = document.createElement('button');
        selectZasilkovnaButton.textContent = 'Vybrat pobočku';
        selectZasilkovnaButton.setAttribute('class', 'packeta-selector-open');
        selectZasilkovnaButton.setAttribute('type', 'button');

        zasilkovnaInput.parentNode.insertBefore(selectZasilkovnaButton, zasilkovnaInput);
        zasilkovnaInput.setAttribute('class', 'packeta-selector-branch-name');
        zasilkovnaHidden.setAttribute('class', 'packeta-selector-branch-id');

        zasilkovnaInput.setAttribute('readonly', 'readonly');
        zasilkovnaInput.parentNode.style.display = 'flex';

        var showHideZasilkovna = function(value) {
            var state = value === zasilkovnaValue ? 'block' : 'none';
            zasilkovnaParent.style.display = state;
            zasilkovnaInput.value = state === 'block' ? '' : 'N/A';
        }

        var transportSelectors = document.querySelectorAll("input[name='" + transportSelectorName + "']");
        for (var i = 0; i < transportSelectors.length; i++) {
            transportSelectors[i].addEventListener('change', function() {
                showHideZasilkovna(this.value);
            });
            if ((transportSelectors[i].value === zasilkovnaValue) && !transportSelectors[i].checked) {
                zasilkovnaParent.style.display = 'none';
            }
        }

        // Po úspěšné inicializaci spustit widget Packeta
        setTimeout(initPacketaWidget, 500);
    }

    // Spustit inicializaci po načtení stránky
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', function() {
            setTimeout(initZasilkovna, 1000);
        });
    } else {
        setTimeout(initZasilkovna, 1000);
    }
</script>

<script>
    // Konfigurace pro nový widget v6
    var packetaOptions = {
        language: 'cs',
        country: 'cz',
        vendors: [
            {
                country: 'cz',
                group: ''  // prázdné pro normální výdejny
            }
        ]
    };
    
    // Inicializace nového widgetu
    function initPacketaWidget() {
        var buttons = document.querySelectorAll('.packeta-selector-open');
        var branchNameInput = document.querySelector('.packeta-selector-branch-name');
        var branchIdInput = document.querySelector('.packeta-selector-branch-id');
        
        // Zkontrolovat, zda prvky existují
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

## Vylepšení v této verzi

- **Responzivní mobilní zobrazení**: Widget se správně přizpůsobí velikosti mobilního zařízení
- **Čekání na načtení**: Kód čeká na kompletní načtení SimpleShop formuláře před inicializací
- **Widget v6**: Používá nejnovější verzi Packeta widgetu s lepší kompatibilitou
- **Vylepšené CSS**: Zajišťuje správné zobrazení na všech zařízeních

## Závěrem

Po uložení by měla integrace fungovat na desktopu i mobilních zařízeních. Postup je třeba zopakovat pro každý produkt v Simpleshopu, který lze zasílat přes Zásilkovnu. Pokud potřebujete pomoc s nastavením, ozvěte se mi (https://milosturek.cz).

Skript umožňuje pouze výběr pobočky - zásilku v zásilkovně si stále musíte vytvořit ručně. Pokud jich máte více, Zásilkovna umožňuje hromadný import z CSV, a Simpleshop zase export objednávek do CSV. Před prvním importem je nutné si v Zásilkovně vytvořit šablonu pro import ze Simpleshopu.

## Řešení problémů

Pokud widget nefunguje:
1. Zkontrolujte správnost všech konfiguračních parametrů
2. Ověřte, že je API klíč platný
3. Zkontrolujte konzoli prohlížeče (F12) pro chybové hlášky
4. Ujistěte se, že je Zásilkovna vybrána jako způsob dopravy