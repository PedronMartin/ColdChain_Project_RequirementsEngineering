Entità e nodi:

    IntegrityVerifier

    SecureElement (rappresentato con icona utente/attore)

    PhysicalTransducer (rappresentato con icona utente/attore)

    APIManager

    DashboardManager

    DeviceManager

    OnBoardFirmware (rappresentato con icona utente/attore)

    AppGateway

    ShipmentManager

Relazioni e flussi (con etichette dei dati associati e direzioni delle frecce)

    SecureElement -> signatureValue -> IntegrityVerifier

    PhysicalTransducer -> measuredValue -> SecureElement

    IntegrityVerifier -> integrityStatus -> DashboardManager

    PhysicalTransducer -> _measuredValue -> DashboardManager

    PhysicalTransducer -> measuredValue, TimeStamp -> APIManager

    OnBoardFirmware -> isoutofrange, batteryLevel -> DashboardManager

    OnBoardFirmware -> BLEConnectionStatus -> AppGateway

    ShipmentManager -> samplingInterval, metricName, minValue, maxValue -> OnBoardFirmware

    ShipmentManager -> logicalState -> APIManager

    AppGateway -> logicalState, tracked_by (instance) -> ShipmentManager

    OnBoardFirmware -> batteryLevel, BLEConnectionStatus -> DeviceManager