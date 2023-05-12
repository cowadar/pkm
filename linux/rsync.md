# Rsync

## Limiteer resource gebruik tijdens Rsync

Je kan de prestaties van rsync verbeteren door enkele opties te gebruiken:

- U kunt de vlag `-z` weglaten als u bestanden lokaal kopieert, aangezien compressie niet nodig is.
- U kunt de vlag `-W` gebruiken om hele bestanden over te zetten zonder de prescan, wat sneller kan zijn voor kleine bestanden of bestanden die veel zijn veranderd.
- U kunt de vlag `--delete` gebruiken om bestanden op de bestemming te verwijderen die niet op de bron staan, wat ruimte en tijd kan besparen.

Hier is een voorbeeld van een aangepaste rsync-opdracht:

```bash
rsync -avhW --delete --exclude='.git/' bedar@10.10.10.67:/home/bedar/fivem/havenstad/resources ./resources
```