---
title: "IQFA'XII - Poster presentation: A versatile PIC-based CV-QKD receiver"

event: IQFA'XII - 12th colloqium on Quantum Engineering, Fundamental Aspects to Applications
event_url: https://iqfacolloq2021.sciencesconf.org/

location: Lyon
address:
  street: Place de l'École
  city: Lyon
  postcode: '69000'
  country: France

summary: IQFA'XII - 12th colloqium on Quantum Engineering, Fundamental Aspects to Applications


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2021-11-03"
date_end: "2021-11-05"
all_day: false

# Schedule page publish date (NOT talk date).
publishDate: "2022-08-30T00:00:00Z"

authors: [nanoy]
tags: []

# Is this a featured talk? (true/false)
featured: false

image:
  focal_point: center

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

---

Quantum Key Distribution is a prominent application of quantum cryptography, offering security based on the laws of quantum physics. Commonly used discrete-variable (DV) QKD requires components such as single-photon detectors that are costly and require specific technology. Continuous-Variable Quantum Key Distribution (CV-QKD) on the other hand can address this issue as it only requires standard telecom laser sources and receivers. [1].  
In order to reduce the cost and size of CV-QKD systems, there are significant efforts to integrate such systems on Photonic Integrated Circuits (PICs) [2]. In particular our team is part of the European quantum technologies flagship project CiViQ [3] where one of the main goals is to demonstrate fully PIC-based CV-QKD systems.  
We report here on the characterization and system integration of two receiver (Rx) PICs (fig. 1a) and their specifically designed low-noise transimpedance amplifier (TIA). The chips were designed by CNRS/C2N and fabricated using a SiGe process by CEA/LETI, and each one can be used as a single-polarization homodyne or heterodyne detector. With an external Polarization Beam Splitter (PBS), we could use two chips as a full double-polarization receiver.  
The results that we obtained demonstrate that the component characteristics relevant to CV-QKD, namely the clearance (shot-noise to electronic noise ratio), the linearity and the quantum efficiencies, on a band of several tens of MHz, are in the required range for CV-QKD and promising for a key exchange. A signal at the quantum levels required for the key exchange was also detected by the chips (fig. 1b). A special care was taken to find and eliminate unwanted sources of noise and get the best conditions for characterizing and using the PICs. We optimized the coupling to get the highest overall quantum efficiencies and the PICs internal Variable Optical Attenuators (VOAs) were also used to get the best clearance possible without saturation.  
The system integration was done with a bulk transmitter (Tx) with several features akin to standard telecom technology including working with continuous wave (CW) and standard Digital Signal Processing (DSP) algorithms [4].  
Near-term future work includes full operation with signal transmission and DSP, and system integration with other PICs of the CiViQ project with a projected Secret Key Rate (SKR) of the order of
kbits/s over metropolitan area distances (fig. 1c).

[1] P. Jouguet, S. Kunz-Jacques, A. Leverrier, P. Grangier, E. Dia-
manti, Nature Photon. 7, 378 (2013).  
[2] M. Ziebell, M. Persechino, N. Harris, C. Galland, D. Marris-
Morini, L. Vivien, E. Diamanti, and P. Grangier, 2015 European
Conference on Lasers and Electro-Optics - European Quantum Electronics Conference, (Optical Society of America, 2015)  
[3] CiViQ project. https://civiquantum.eu  
[4] Roumestan et al., ECOC 2021
