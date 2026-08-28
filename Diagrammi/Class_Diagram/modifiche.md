
gli id vanno bene (shipment e hw) perchè non sono riferimenti/Link ad altre entità ma sono oggetti logici che identificano l'entità stessa al suo stesso interno.

La chiave pubblica è stata spostata dal generatore crittografico al iot device perchè senno vorrebbe dire che ogni volta la crittografia usa la stessa chiave per ogni dato/sensore; la chiave privata invece non è un qualcosa che modelliamo in quanto va mantenuta segreta.

il timestamp samplingInterval è stato cambiato in Integer (non c'era timestamp come cast), in questo modo rappresentiamo direttamente il numero di secondi di intervallo.

I timestamp invece che rappresentavano un momento esatto ho messo Date.

Nome di alarmType modificato in alarm perchè il tipo era già AlarmType ed erano troppo simili.