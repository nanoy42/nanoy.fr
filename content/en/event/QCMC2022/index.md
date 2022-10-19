---
title: "QCMC2022 - Poster presentation: A versatile PIC-based CV-QKD receiver"

event: International Conference on Quantum Communication Measurement and Computing (QCMC) 2022
event_url: http://www.qcmc-lisbon.org/
location: Lisbon
address:
  street: 45A Avenida de Berna
  city: Lisbon
  postcode: '1050-078'
  country: Portugal

summary: International Conference on Quantum Communication Measurement and Computing (QCMC) 2022


# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
date: "2022-07-11T12:00:00Z"
date_end: "2022-07-12T12:00:00Z"
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

Quantum Key Distribution (QKD) is a stepping stone of quantum cryptography and allows two distant parties, Alice and Bob, to share a secret key with a security based on the laws of quantum physics. Contrary to commonly used Discrete-Variable (DV) systems, which require specific components such as single-photon detectors, Continuous-Variable (CV) QKD allows for the use of standard telecom devices and can also improve the secret key rates [1].  
To reduce the cost and size of CV-QKD systems, significant efforts have been made towards monolithic integration on Photonic Integrated Circuits (PICs) [2,3]. In this context, we are involved in the European quantum technologies flagship project CiViQ [4], where a central goal is the demonstration of a fully PIC-based CV-QKD system. To this end, we have designed and characterized a PIC receiver and its ultra-low noise readout electronics and have used it together with a bulk transmitter for overall system assessment.  
The PIC, which mainly includes beam splitters, balanced photodetectors and variable optical attenuators (VOAs), was manufactured in the CEA/LETI silicon germanium process. The electronics, which mainly includes a transimpedance amplifier, a V/V amplifier and filters, was manufactured using carefully selected commercial components. In this work, we have characterized the PIC performance as a single-polarization heterodyne detector. The critical characteristics are the clearance (electronic-noise-to-shot-noise ratio), the quantum efficiency, the linearity and the bandwidth. We have shown that these are in the suitable ranges for CV-QKD. The on-chip VOAs were tuned for a perfect balance of the detection avoiding electronics saturation and therefore improving detection performance.  
The PCB (chip and electronics, see fig. 1) was subsequently used in a CV-QKD system, together with a bulk transmitter. We processed the exchanged data using specifically designed software and Digital Signal Processing (DSP), which allows to recover and process, on Bob’s side, the symbols sent by Alice [5]. The DSP includes synchronization, clock recovery, phase correction and symbol recovery.  
Near-term future work includes improving the DSP for a full CV-QKD transmission and integration with other PIC-based subsystems, in particular the transmitter including a quantum random number generator.

[1] E. Diamanti and A. Leverrier, Entropy 17, 6072 (2015).  
[2] M. Ziebell et al., EQEC-CLEO Europe 2015.  
[3] G. Zhang et al., Nature Photon. 13, 839 (2019).  
[4] CiViQ project. https://civiquantum.eu  
[5] F. Roumestan et al., ECOC 2021.