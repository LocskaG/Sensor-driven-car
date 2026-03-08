# Sensor-driven-car
Hardverarchitektúra

A rendszer felépítése a hatékony erőforrás-gazdálkodásra törekszik, különös tekintettel az Arduino Uno korlátos I/O lábszámára.
Főbb komponensek:

    Vezérlő: Arduino Uno R3 + Sensor Shield V5.0

    Meghajtás: L298N (vagy L293D) H-híd motorvezérlő + 4 db DC motor

    Érzékelés:  
        - HC-SR04 Ultrahangos távolságmérő (szervóra rögzítve)
        - Infravörös (IR) vevőegység
        - Optikai vonalkövető szenzor
