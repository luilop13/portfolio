Research
========

Research Projects
------------------

Within this section you can find a summary of the research projects I have been involved in during my academic career.
These proyects are ordered from the most recent to the oldest.

Automatic Recognition and Analysis of Collapsing Discharges in NBI Plasmas in the TJ-II Stellarator.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This research was conducted as part of my collaboration with the CIEMAT to develop my master’s thesis regarding nuclear fusion and machine learning within the framework of the `EUROfusion project <https://euro-fusion.org/eurofusion/members/spain/>`_.
To explain the context of this work, a brief introduction to nuclear fusion and the TJ-II stellarator is presented below.

Nuclear fusion is the process of combining nuclei of light atoms into heavier ones (see :numref:`fig-dt-fusion`).
This reaction occurs at very high temperature in the fourth state of matter, plasma, a highly ionized gas where electrons are free.

.. figure:: /_static/images/fusion_reaction.png
   :name: fig-dt-fusion
   :width: 60%
   :align: center

   Deuterium-tritium (D-T) fusion reaction.

To study plasma, there are two main magnetic confinement devices: tokamak and stellarator (see :numref:`fig-fusion-devices`). This work was developed at TJ-II.

.. figure:: /_static/images/fusion_devices.png
   :name: fig-fusion-devices
   :width: 90%
   :align: center

   Magnetic confinement fusion devices: Tokamak (left) and Stellarator (right).

Plasma is achieved at very high temperatures. For this purpose, TJ-II has two heating systems: one that heats the plasma with microwaves tuned to the resonance frequency of electrons, and another with high-power neutral beam injections (see :numref:`fig-heating-systems`).
When heating occurs, what is called a discharge is generated, which is the period of time in which the plasma is born, lives, and dies.

.. figure:: /_static/images/heating_tjii.png
   :name: fig-heating-systems
   :width: 35%
   :align: center

   TJ-II heating systems, which consists of two gyrotrons, ECH1 and ECH2, the power is transmitted to the plasma by two quasi-optical transmission lines. And two neutral beam injectors, NBI1, the co-injector, and NBI2, counter-injector.

The TJ-II database consists of 60,000 discharges, and each one contains around 500 signals. In this work, 6 of these signals have been used (see :numref:`fig-shot_56036`):

* The heating signals, NBI1 and NBI2, which indicate whether the two neutral beam injection systems are acting or not. When the last NBI signal of the two drops to zero, heating is considered to have ended.
* The electron density signal in red.
* The plasma energy signal in green.
* A radiation signal in blue.

.. figure:: /_static/images/shot_56036.jpg
   :name: fig-shot_56036
   :width: 55%
   :align: center

   Critical TJ-II signals for collapse diagnosis.

In discharges, the desirable operation is when the plasma energy signal begins to decay when heating ends.
But there are occasions when a radiative-type collapse occurs, that is, an impurity enters the plasma, it loses energy and cools down before heating ends. As can be seen in :numref:`fig-shots-comparison`, the energy signal decays and dies before heating ends.
This behavior is anomalous and undesirable, so it is essential to find a method to classify discharges as collapsing and non-collapsing.

.. figure:: /_static/images/shots_comparison.png
   :name: fig-shots-comparison
   :width: 90%
   :align: center

   Collapsing vs non-collapsing discharges. The collapsing shot’s energy signal decays before the end of the heating, determined by both NBIs. In the non-collapsing case, the energy signal has not finished its decay before the end of the heating.

There are occasions when classification is very complicated because there are doubtful discharges. In this case, the energy signal decays before heating ends, but does not reach zero until after heating ends. Therefore, the binary classifier that will be implemented must learn from the behavior of the signals when the discharge is collapsing and when it is not.

Previously, at LNF, this classification was done manually. Consequently, the objective of this work is to automate the processing and classification of collapsing discharges from the database of 60,000 discharges. For this purpose, a MATLAB-based application called Plasma Collapse Analysis Identifier Tool, PCAI Tool, has been created. The collapse phenomenon is not explained from a physical point of view, so it is crucial to analyze the maximum number of discharges possible with this tool.

On the other hand, plasma electron density signals normally have the shape on the left in :numref:`fig-density-comparison`. But there are occasions when this type of signal presents a phenomenon called fringe jump. As can be seen, the red signal presents this phenomenon, which are steps that totally distort the signal value. If we want to measure the density at the moment of collapse, we have to reconstruct it. For this purpose, an algorithm has been developed that detects these jumps and reconstructs the signal by splicing the sections. The result is the signal in green.

.. figure:: /_static/images/density_comparison.png
   :name: fig-density-comparison
   :width: 100%
   :align: center

   Regular density signal (left, green) VS density signal with fringe jump (right, red) and reconstructed signal (right, green).

Regarding the method for discharge classification, a Support Vector Machine has been chosen, whose acronym is SVM. When training an SVM, feature vectors are introduced with a label manually assigned by the user. Then, the SVM generates a hyperplane that separates the two classes. A linear kernel is used. To predict new labels, feature vectors without labels are introduced into the trained SVM and the model automatically assigns them.

In the PCAI Tool, for classifying collapsing discharges, the user can choose which signal to use and the sample size starting from the end of heating. In the example, it is 1024 and this dimensionality will be reduced with the wavelet transform, whose decomposition level is also configurable.
To classify density signals between anomalous and normal, a fixed sample size will be used, and when applying the transform, the user can choose the decomposition level and the type of coefficient, whether approximation or detail.

Having said that, the PCAI Tool consists of the functional modules illustrated in :numref:`fig-sw-arch`.

.. figure:: /_static/images/sw_arch.png
   :name: fig-sw-arch
   :width: 100%
   :align: center

   PCAI Tool architecture.

* **Download Module**, downloads the signals of each discharge from the Web server
  where the TJ-II discharges are stored, and passes the downloaded information to the
  Signal Module.


* **Signal Module**, integrates signal processing algorithms and stores each
  discharge with its corresponding signals in the "Temporal Data" folder, in a ".mat"
  file format. Shots within these files must be assessed by the user through the GUI.
  Once they are evaluated, by a command of the user interface they are passed from the
  temporary folder to the definitive one, "Validated Data". Discharges within the latter
  folder are used for further operations.


* **Data Module**, stores and retrieves information from the database and the discharges
  available in the validated data folder. It communicates with the user through the GUI
  to store the discharges and density signals’ labels assigned by the expert system, i.e.
  the user. If necessary, communicates back with the signal module for further signal
  processing. All modules that require data retrieval are connected to this one.


* **SVM Module**, generates the feature vectors to train the SVM models and saves them
  in a ".mat" file within the "Training Files" folder. With those files, the module trains
  SVM models and saves them in a ".mat" file within the "SVM Models" folder.


* **Test Module**, uses an SVM model to test it with labeled discharges or density signals.
  Results are stored in a ".mat" file within the "Test Files" folder.


* **Prediction Module**, uses an SVM model to predict unlabeled discharges or density
  signals. Results are stored in a ".mat" file within the "Prediction Files" folder.


* **Analysis Module**, uses the Prediction Model files and the manually labeled data
  retrieved by the Data Module to perform further analysis, which is output through the
  user interface.

:numref:`fig-unkown-labels` shows one of the many interfaces of the PCAI Tool, where the  user
can classify discharges automatically with a trained SVM model. 

.. figure:: /_static/images/unknown_labels.png
   :name: fig-unkown-labels
   :width: 60%
   :align: center

   PCAI Tool discharge classification interface.

Regarding classification results, two main measures are used:

* **Accuracy**, which is the total number of correct predictions with respect to the total number of discharges.
* **F1-score**, which is a more robust measure if the number of collapsing and non-collapsing discharges is disproportionate.

For discharge classification, an accuracy of 96.4% and an F1-score of 93.75% have been obtained. And 
for density signal classification, an accuracy of 73.26% and an F1-score of 80.67% have been obtained.

Thanks to the development of this work, the following goals have been met, which are of 
great importance to the LNF final users:

* Automated the analysis of the signals coming from the TJ-II.

* Automated collapsing/not collapsing discharge classification on the TJ-II database of 
  60000 shots.

* Compiled the PCAI Tool, so that every final user of the LNF could easily install it 
  without the need of a MATLAB license.

* Developed the density signal reconstruction algorithm, palliating the fringe jump phenomenon, 
  which happens in interferometry diagnostics, setting an unprecedented approach.

* Implemented user-configurable installation settings for signal download, allowing future 
  updates and the inclusion of new signals.

* Created an SVM classifier with an accuracy of 96.4% and an *F*\ :sub:`1` *-score* of 
  93.75%.

This work has contributed to the `Plasma 2025 <https://plasma2025.ipplm.pl/wp-content/uploads/2025/09/Agenda_PLASMA2025.pdf>`_  – International Conference on Research 
and Application of Plasmas. The title of the contribution is: Recognition of two patterns of ion-temperature profiles in the TJ-II stellarator after 
the upgrade of a multichannel spectrometer.

Test-bench Design for Silicon Strip Radiation Detectors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As a recipient of the Ministry of Education Collaboration Grant, I worked with the University of Huelva's 
Department of Electrical and Thermal Engineering, Design and Projects on nuclear physics research within 
the framework of the `G.R.I.T. project <https://grit.in2p3.fr/>`_, in the LIFE Laboratory (Laboratorio de 
Interacciones FundamentalEs, which means laboratory of fundamental interactions) as an Undergraduate Research 
Assistant.

The aim of this work was to design a test-bench to characterize strip detectors
for the G.R.I.T. collaboration. A typical test-bench should allow to obtain both:
(i) the IV curves and (ii) the energy spectrum of the detector under study.
This aim was achieved by tackling the following objectives:

* Design an electrical interface to extract the signal from the detector.
* Design of a polarization circuit.
* Design of a radiation source holder.
* Measure the I-V curve.
* Measure the radiation of the triple-alpha source.

The setup employed to obtain the I-V curves is shown in :numref:`fig-iv-setup`, where all
the equipment needed are also presented. The general purpose of this setup
is to polarize all the strips at the same time, for different voltages, while
measuring the current, which is the total leakage current of the detector.

.. figure:: /_static/images/iv_setup.jpg
   :name: fig-iv-setup
   :width: 80%
   :align: center

   I-V curve setup diagram. The whole detector is polarized to obtain the total leakage current.

As a result of the setup, :numref:`fig-iv-curve` shows the I-V curves obtained in three different scenarios: (i)
the orange curve (bottom) was given by the detector manufacturer, (ii) the
red curve (middle) was obtained in the lab employing the source meter and
(iii) the blue curve (top) was done manually using the MHV-4.

.. figure:: /_static/images/iv_curve.png
   :name: fig-iv-curve
   :width: 60%
   :align: center

   Total leakage current (i.e. all strips polarized) with detector in a 10\ :sup:`-5`\  mbar vacuum. The bottom curve is the one provided by the manufacturer.

The alpha spectroscopy setup (see :numref:`fig-alpha-setup`) is more complex as the signals
from several strips have to be processed. In the diagram shown in :numref:`fig-alpha-setup`, the alpha
source, that should face the detector, has been omitted.

.. figure:: /_static/images/alpha_setup.jpg
   :name: fig-alpha-setup
   :width: 80%
   :align: center

   Alpha spectroscopy setup diagram. With this setup all strips can be read.

The output of the spectroscopy setup is shown in :numref:`fig-energy-spectrum`, where the energy
spectrum from a triple-alpha source is depicted.

.. figure:: /_static/images/energy_spectrum.png
   :name: fig-energy-spectrum
   :width: 60%
   :align: center

   Energy spectrum from a triple-alpha source depicting the :sup:`239`\Pu, :sup:`241`\Am and :sup:`244`\Cm peaks, and their energies.

Finally, thanks to this work, I could write my Bachelor's thesis entitled "Test-bench for strip radiation detectors:
design & experimental results" and this test-bench was installed in the INFN (Instituto Nazionale di Fisica Nucleare) in Legnaro, Italy,
from where we obtained the experimental results that allowed us to publish the first paper I participated in :cite:`s23125384`.

.. bibliography::