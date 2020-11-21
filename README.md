<html><head><meta content="text/html; charset=UTF-8" http-equiv="content-type"></head><body><div class="c31 c50"><h1 class="c45" id="h.v0rgan1jjm8x"><span class="c35">Heart Rate Detection Using Remote Photoplethysmography</span></h1><p class="c8"><span class="c40">David Hass, Spencer Mullinix, Hogan Pope</span></p><p class="c8"><span class="c27">Fall 2020 ECE 4554/5554 Computer Vision: Course Project</span></p><p class="c8"><span class="c38">Virginia Tech</span><hr><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 228.50px; height: 198.36px;"><img alt="" src="images/image4.png" style="width: 228.50px; height: 198.36px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></p><h3 class="c26" id="h.itx3moozawen"><span class="c23">Abstract</span></h3><p class="c8"><span class="c14">Currently heart rate is an attribute that can be incredibly difficult to measure without being in close proximity to the patient. However, by using modern Computer Vision techniques, a close approximation to heart rate can be discovered with nothing more than a live video feed. This paper details a remote photoplethysmography implementation utilizing Fourier techniques and Independent Component Analysis to estimate a subject&rsquo;s BVP signal, and furthermore their heart rate. We report moderate success, with an error of </span><span class="c36 c46">[INSERT ERROR HERE]</span><sup><a href="#cmnt1" id="cmnt_ref1">[a]</a></sup><span class="c7">. Further work should explore real-time implementations of our algorithm, along with reducing the use of priors within our work.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8 c9"><span class="c7"></span></p><h3 class="c26" id="h.xrf0xh2mhh1x"><span class="c24">Introduction</span></h3><p class="c8"><span class="c7">Finding a client&#39;s heart rate either for health, polygraph, or other reasons, is an issue that classically requires close proximity. However, with recent developments in remote photoplethysmography, this is no longer necessarily the case. If a stable version of this were deployed, it would allow for better remote healthcare work and improvements in other areas where being remote can help lower costs or increase availability. Our work only necessitates an RGB camera capable of recording video, as part of our goal is to make this capability available to as wide an array of people as possible. Hopefully being able to make them deployable on nearly all modern laptops, as well as potentially smartphones. One of the ways this issue has been approached in the past, specifically in the realm of smart phones, is through the use of fingerprint scanners. However, one of the benefits of being able to do this entirely via camera, is that while fingerprint scanners are becoming increasingly common in smartphones, they are all but non-existent in laptops, and many other devices that already have integrated cameras. Thus using only a camera would increase the domain of devices that could be supported.</span></p><p class="c8 c9"><span class="c7"></span></p><h3 class="c26" id="h.7d7oyvfa4dm0"><span class="c24">Approach</span></h3><p class="c8"><span class="c7">We implemented an image processing pipeline aimed towards extracting a subject&#39;s blood volume pulse (BVP) signal, and from that, their pulse rate with a technique called remote photoplethysmography (rPPG). The algorithms are fed a video of a subject, and processed each frame of the video to extract time-indexed RGB vectors. The vectors then go through a pipeline of spectral and statistical analysis algorithms to extract a BVP signal, and from that, their heart rate.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c14">The spectral method we have implemented is inspired by Poe et al. [1] and consists of roughly three portions: ROI detection, preprocessing and extraction, and pulse rate calculation. The first of which is aimed to calculate the location of the subject&#39;s face to measure the BVP signal. To maintain a robust sequence of measurements on a relatively static portion of the subject&rsquo;s face, we used the DLib facial landmark detector to extract the area between the subject&rsquo;s cheeks [2], illustrated in </span><span class="c14 c37">Figure 6</span><span class="c14">. After their face has been segmented, each RGB channel in the face-image is averaged, resulting in one measurement per channel per frame in the video, producing three signals. These signals are illustrated in </span><span class="c14 c37">Figure 1</span><span class="c7">. </span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c14">As heart rate signals are non-stationary, we then detrended these signals using a smoothness priors approach (</span><span class="c14 c37">Figure 2</span><span class="c7">) with a cutoff frequency of 0.33 Hz [3]. After the RGB signals have been detrended and z-normalized, we use Independent Component Analysis (ICA) to decompose them into three independent source signals, shown in figure 3. To ensure that we could perform this step robustly and quickly, we opted to use scitkit-learn&rsquo;s FastICA implementation [4]. ICA separates color variations due to BVP from variations caused by motion, lighting, or other sources. One of the returned components represents the fluctuations in color caused by variations in blood volume; this is assumed to be the component with the largest peak in its power spectrum. An example of an extracted BVP signal is shown in figure 4.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c7">We then filter the signal in the time and frequency domains with a 5 point moving-average filter and a hamming window bandpass filter with cut-off frequencies depending on the user&rsquo;s inputted state. We allow them to choose between resting, recovery, and active; each of which have different cut-off frequencies that incorporate prior estimates of their heart rate. Once the BVP signal is calculated, we use the interbeat-interval estimation implementation described in van Gent et al [5] to estimate the heart rate of the subject. We decided to utilize these authors&rsquo; implementations because the core focus of our project is on remotely estimating the BVP signal, not interbeat-interval estimation.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8 c9"><span class="c7"></span></p><h3 class="c26" id="h.wrh8w477mucc"><span class="c49">Experiments</span><sup><a href="#cmnt2" id="cmnt_ref2">[b]</a></sup></h3><p class="c8"><span class="c7">We compared our methods to other popular methods of RPPG(Remote Photoplethysmography) as seen in the table below [5].</span></p><p class="c8"><span class="c7">In this table, the values listed in each cell is the Signal-to-Noise-Ratio (SNR).</span></p><a id="t.5b683a68fc0c97676194c68eca10073f738de906"></a><a id="t.0"></a><table class="c18"><tbody><tr class="c47"><td class="c5" colspan="1" rowspan="1"><p class="c3"><span class="c0">Category</span></p></td><td class="c12" colspan="1" rowspan="1"><p class="c3"><span class="c0">Challenge</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c3"><span class="c0">G(2007)</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c3"><span class="c0">G(2008)</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c3"><span class="c0">PCA(2011)</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c3"><span class="c0">ICA(2011)</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c3"><span class="c0">CHROM(2013)</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c3"><span class="c0">PBV (2014)</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c3"><span class="c0">2SR (2014)</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c3"><span class="c0">Spectral Method</span></p><sup><a href="#cmnt3" id="cmnt_ref3">[c]</a></sup></td></tr><tr class="c19"><td class="c5" colspan="1" rowspan="3"><p class="c8"><span class="c7">Skin Type</span></p></td><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Type I-II</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">2.67</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">7.55</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">5.85</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.51</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.47</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">5.57</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">7.44</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">TBD</span></p><sup><a href="#cmnt4" id="cmnt_ref4">[d]</a></sup></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Type III</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">2.07</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">7.89</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">5.38</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.61</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.21</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.26</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">7.90</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p><sup><a href="#cmnt5" id="cmnt_ref5">[e]</a></sup></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Type IV-V</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-0.49</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.40</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">2.25</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.56</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">5.43</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.04</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.60</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p><sup><a href="#cmnt6" id="cmnt_ref6">[f]</a></sup></td></tr><tr class="c19"><td class="c5" colspan="1" rowspan="3"><p class="c8"><span class="c7">Luminance</span></p></td><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Stationary</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">8.10</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">10.14</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">8.70</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">11.61</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">9.42</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.57</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">10.53</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">TBD</span></p><sup><a href="#cmnt7" id="cmnt_ref7">[g]</a></sup></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Rotation</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">0.81</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.34</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">1.46</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.04</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.63</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.36</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">6.16</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p><sup><a href="#cmnt8" id="cmnt_ref8">[h]</a></sup></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Talking</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-0.62</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.75</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">0.46</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.11</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.99</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.01</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">5.33</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p></td></tr><tr class="c19"><td class="c5" colspan="1" rowspan="3"><p class="c8"><span class="c7">Recovery</span></p></td><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Low</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-3.07</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.67</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">-0.60</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">1.78</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">2.66</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">1.95</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.93</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">TBD</span></p></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Medium</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-3.19</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.97</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">-0.79</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">1.65</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.62</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.15</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">5.26</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">High</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-8.19</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.11</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">-6.51</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">-0.82</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.52</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.52</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.84</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p></td></tr><tr class="c19"><td class="c5" colspan="1" rowspan="2"><p class="c8"><span class="c7">Skin Type</span></p></td><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Biking</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-6.39</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">-3.38</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">-4.21</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">-5.50</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">0.68</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">0.57</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">-0.28</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p></td></tr><tr class="c19"><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Stepping</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-12.59</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">-9.06</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">-11.41</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">-12.51</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">-3.13</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">-2.85</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">-4.50</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">N/A</span></p></td></tr><tr class="c19"><td class="c5" colspan="1" rowspan="1"><p class="c8"><span class="c7">Overall</span></p></td><td class="c12" colspan="1" rowspan="1"><p class="c8"><span class="c7">Average</span></p></td><td class="c11" colspan="1" rowspan="1"><p class="c8"><span class="c7">-1.90</span></p></td><td class="c13" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.67</span></p></td><td class="c2" colspan="1" rowspan="1"><p class="c8"><span class="c7">0.05</span></p></td><td class="c17" colspan="1" rowspan="1"><p class="c8"><span class="c7">1.92</span></p></td><td class="c21" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.86</span></p></td><td class="c4" colspan="1" rowspan="1"><p class="c8"><span class="c7">3.56</span></p></td><td class="c25" colspan="1" rowspan="1"><p class="c8"><span class="c7">4.93</span></p></td><td class="c15" colspan="1" rowspan="1"><p class="c8"><span class="c7">TBD</span></p></td></tr></tbody></table><h4 class="c28 c29" id="h.jeilyf5a453g"><span class="c39 c43"></span></h4><h4 class="c28" id="h.1n85odtksc1x"><span class="c34">Data Sets</span></h4><p class="c8"><span class="c16"><a class="c30" href="https://www.google.com/url?q=https://mahnob-db.eu/hci-tagging/&amp;sa=D&amp;ust=1605903631393000&amp;usg=AOvVaw3FXTrfgvC5-ZQ22SBtDpYZ">HCI tagging database</a></span><span class="c7">: This database is videos and images of 30 test subjects&#39; biometric reactions to stimuli. This dataset includes images and biometric data for these subjects.</span></p><p class="c8"><span class="c16"><a class="c30" href="https://www.google.com/url?q=https://osf.io/fdrbh/wiki/home/&amp;sa=D&amp;ust=1605903631393000&amp;usg=AOvVaw2NZniIJyalrdpc2KbDwOSW">OSF rPPG</a></span><span class="c7">: This dataset covers provide RGB images and videos that are tagged with the foreground and background of the image as well as the biometrics of the people in the image.</span></p><p class="c8"><span class="c16"><a class="c30" href="https://www.google.com/url?q=https://www.idiap.ch/dataset/cohface&amp;sa=D&amp;ust=1605903631393000&amp;usg=AOvVaw2jJ05EP_6Yd8Vk6vJ1fsLp">COHFACE dataset</a></span><span class="c7">: This dataset consists of 160 minutes of 40 individuals of varying genders with tagged biometric data. The only downside to this database is that we will need assistance gaining access.</span></p><p class="c8"><span class="c7">Collected Data: Eleven videos were taken of ten different people in ten different lighting conditions. Their heart-rates were taken using established ppg algorithms. These readings were used as ground truth for training.</span></p><p class="c8 c9"><span class="c7"></span></p><h4 class="c28" id="h.hrzisetzxl4d"><span class="c34">Experimental Methodology</span></h4><p class="c8"><span class="c7">Our testing approach consists of two phases of testing. The first phase consists of preliminary tests that test the initial operating capability of our project. Phase one tests the algorithm on data of people of the same skin tone, gender, with no facial hair, under ideal lighting conditions, with the test subject facing the camera straight on from a fixed distance. This methodology of testing will allow students to determine the algorithms operating capacity before other variables are introduced. The second phase of testing introduces the variables that were fixed in the first phase. Each variable is changed independently of the rest to isolate that particular variable. This enables students to evaluate the algorithms performance in specific test cases.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c7">This methodology for testing the algorithm allows for complete testing of variables while isolating the faults. The algorithm will be successful if the algorithm can achieve &plusmn;5% in phase one. Success is more difficult to define for the second phase of testing. Success in the second phase will be rated on initial performance versus improvement. If a test case in phase two has &plusmn;10% error then it is considered a success. If the error is greater than &plusmn;10% the success criteria will be defined by the amount of improvement that can be achieved due to tweaking the algorithm.</span></p><p class="c8 c9"><span class="c7"></span></p><h4 class="c28" id="h.58yaoe9kt2dg"><span class="c34">Test Cases</span></h4><p class="c8"><span class="c14">We performed an experiment measuring the effect of subject distance from the camera on the accuracy of the derived PPG signal, compared to a reference measurement from a pulse oximeter sensor. We believe that distance may influence accuracy, due to decreasing pixel density of the face-image as the subject moves further from the camera. To test this, we will record 1 minute videos of the subject 0.5 meters from the camera, 1 meter from the camera, and 2 meters from the camera. Variables such as camera location, lighting, and video quality will be held constant. We expect that the derived PPG signals of further away subjects will have a higher signal-to-noise ratio, and thus be less accurate.</span><sup><a href="#cmnt9" id="cmnt_ref9">[i]</a></sup></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c7">We will also perform an experiment with the attempt to measure the program&#39;s ability to track a single face within a video, regardless of motion, both linear and rotational. This could potentially be one of the most challenging tests the program undergoes, as the heart rate is measured from very minor movements of the face, so moving the face throughout the video could drastically affect the results. As with the experiment listed above, these tests will be performed by recording 1 minute videos of the subject while they move their head at varying rates, the rest of the video, lighting and quality, will remain constant. We expect to see a decrease in the accuracy of the program as the basis of tracking heart rate is based on very precise motion.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c14">An experiment will be performed on the image quality that is necessary to get valid data. One of the main flaws of image processing is the general lack of resistance to noisy data. One of the tests will add noise to the testing images and compare the accuracy. The other image quality test will be image resolution. This will involve downsampling the testing images to different degrees and taking metrics at each step. The expected performance in both of these situations will be decreased accuracy. The goal is to determine how robust the algorithm is. This will be achieved by incrementally testing by adding more noise/downsampling and comparing the results.</span><sup><a href="#cmnt10" id="cmnt_ref10">[j]</a></sup></p><h3 class="c26 c41" id="h.j5q4k54vg0qk"><span class="c24"></span></h3><p class="c8"><span class="c42">Initial Experiments: Facial Recognition</span></p><p class="c8"><span class="c6">&nbsp; &nbsp; The project revolves around facial recognition, so it was imperative that this portion be prioritized. Currently, the software takes the input of a xml data from the HCI Tagging Dataset. This data is then parsed for faces using OpenCV. Then a bounding box is drawn around the center. The center of the face was decided to be used based off of previous projects in the same field as ours. The center of the face seemed to provide good data while limiting the outside factors. Once the bounding box is drawn the average RGB value for each color channel within the box is saved. This process is repeated for every frame of the video. Each color channel is saved off as a signal after the video has finished being processed as shown below.</span></p><p class="c8 c9"><span class="c6"></span></p><p class="c8"><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 473.33px;"><img alt="" src="images/image1.png" style="width: 624.00px; height: 473.33px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></p><p class="c3"><span class="c6">Figure 1 - Raw RGB Signals</span></p><p class="c1"><span class="c6"></span></p><p class="c8"><span class="c42">Initial Experiments: Signal Filtering</span></p><p class="c8 c9"><span class="c6"></span></p><p class="c3"><span class="c6">Detrended RGB signal data</span></p><h3 class="c26" id="h.uzr0qtlnguec"><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 469.33px;"><img alt="" src="images/image2.png" style="width: 624.00px; height: 469.33px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></h3><p class="c3"><span class="c6">Figure 2 - Detrended and normalized RGB signals</span></p><p class="c1"><span class="c6"></span></p><p class="c8"><span>I</span><span>ndependent component analysis (ICA) was performed on the color channel signals. Independent component analysis is a method of taking input signals then breaking them down into the components that make up those signals. The data above labeled &ldquo;Detrended RGB signal data&rdquo; was input into an ICA algorithm and the output is shown below.</span></p><p class="c8 c9"><span class="c6"></span></p><h3 class="c26" id="h.ktwkhy9vr2rl"><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 468.00px;"><img alt="" src="images/image6.png" style="width: 624.00px; height: 468.00px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></h3><p class="c3"><span class="c6">Figure 3 - ICA components</span></p><p class="c1"><span class="c6"></span></p><p class="c8"><span>Below is an ICA component, which our algorithm has determined to be the most likely BVP signal. The signal chosen has the largest magnitude within our bandpass frequencies, discussed in the approach. </span></p><h3 class="c48" id="h.73pdyck9zoe0"><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 220.00px;"><img alt="" src="images/image5.png" style="width: 624.00px; height: 220.00px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></h3><p class="c3"><span class="c6">Figure 4 - The ICA component selected as our BVP Signal</span></p><p class="c1"><span class="c6"></span></p><p class="c8"><span>After a signal is chosen from ICA, a bandpass filter is applied to eliminate high and low frequency noise. This can be done because a heart beat is typically between 0.7-4.0 Hz [1].</span></p><h3 class="c26" id="h.h4iv4bnw3ghg"><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 432.00px;"><img alt="" src="images/image7.png" style="width: 624.00px; height: 432.00px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></h3><p class="c3"><span class="c6">Figure 5 - Bandpass filtered BVP signal</span></p><p class="c1"><span class="c6"></span></p><p class="c8"><span>After running some manual calculations, there are 26 peaks across approximately 1400 points. This produces a heart rate of 66.8 BPM. We have yet to parse out the data from our dataset about the actual heart rate from this subject. However, a measure of 66.8 BPM seems to be very reasonable as a potentially valid result.</span><sup><a href="#cmnt11" id="cmnt_ref11">[k]</a></sup></p><p class="c8 c9"><span class="c6"></span></p><p class="c8 c9"><span class="c6"></span></p><h3 class="c26" id="h.7qznsxyw9pii"><span class="c24">Qualitative results</span></h3><p class="c8"><span class="c7">We detail many of our results in the experiments section. However, there are a few results worth discussing in this section as well.</span></p><p class="c8 c9"><span class="c7"></span></p><p class="c8"><span class="c14">Figure 6 shows a visualization we developed that runs while our algorithm extracts RGB data. The red box shows the ROI over which the average red, blue, and green signals are extracted from. We calculate this ROI through taking 50% of the Haar cascade&rsquo;s facial bounding box result. In further experiments, we hope to eliminate noise introduced by eye movement through using facial keypoint detection to extract ROIs for the cheeks and face.</span><sup><a href="#cmnt12" id="cmnt_ref12">[l]</a></sup></p><p class="c8 c9"><span class="c7"></span></p><p class="c3"><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 433.58px; height: 319.66px;"><img alt="" src="images/image3.png" style="width: 433.58px; height: 319.66px; margin-left: 0.00px; margin-top: 0.00px; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px);" title=""></span></p><p class="c3"><span class="c7">Figure 6 - A screenshot of the algorithm extracting color data from the subject</span></p><p class="c1"><span class="c7"></span></p><p class="c8 c9"><span class="c6"></span></p><h3 class="c26" id="h.86hwqjfzdq5b"><span class="c24">Conclusion</span></h3><p class="c8"><span class="c6">This report has described an overview of our approach for remote photoplethysmography. We utilize a variety of signal processing and statistical techniques to remotely extract a heart rate signal from a patient. The basic pipeline is as follows: extract color signals from a face within a video, filter them and separate them into independent source signals, determine which source signal is the subject&rsquo;s BVP, and estimate their heart rate from that signal. To accomplish this, we wrote a program in Python that incorporates much of our own work mixed with supporting libraries from reputable authors.</span></p><p class="c8 c9"><span class="c6"></span></p><p class="c8"><span class="c6">Although our implementation is complete, it is not perfect. Future work should investigate estimating heart rate without the use of priors about the subject&rsquo;s activity levels, as one&rsquo;s heart rate may not always fall within the bounds suggested by their activity level. Furthermore, our algorithm processes video at about 3 frames per second, so it would be difficult to implement it online. Real-time heart rate estimates are of much use to physicians and other concerned parties, so this would be a useful expansion of our work.</span></p><p class="c8 c9"><span class="c6"></span></p><p class="c8 c9"><span class="c7"></span></p><h3 class="c26" id="h.ot41mmqx0vp2"><span class="c24">Citations</span></h3><p class="c8"><span class="c7">1. Poh, M.-Z., McDuff, D.J., Picard, R.W.: Advancements in noncontact, multiparameter physiological measurements using a webcam. IEEE Trans. Biomed. Eng. 58, 7&ndash;11 (2011)</span></p><p class="c8"><span class="c14">2. </span><span class="c14 c31">Davis E. King. </span><span class="c14">Dlib-ml: A Machine Learning Toolkit</span><span class="c14 c31">. </span><span class="c14 c37">Journal of Machine Learning Research</span><span class="c14 c31">&nbsp;10, pp. 1755-1758, 2009</span></p><p class="c8"><span class="c7">3. M. P. Tarvainen, P. O. Ranta-Aho, and P. A. Karjalainen, &ldquo;An advanced detrending method with application to HRV analysis,&rdquo; IEEE Trans. Biomed. Eng., vol. 49, no. 2, pp. 172&ndash;175, Feb. 2002.</span></p><p class="c8"><span class="c14">4. </span><span class="c14 c31">Scikit-learn: Machine Learning in Python</span><span class="c10">, Pedregosa </span><span class="c10 c37">et al.</span><span class="c10 c39">, JMLR 12, pp. 2825-2830, 2011.</span></p><p class="c8"><span class="c10">5. </span><span class="c36 c31">van Gent, P., Farah, H., van Nes, N. and van Arem, B., 2019. Analysing Noisy Driver Physiology Real-Time Using Off-the-Shelf Sensors: Heart Rate Analysis Software from the Taking the Fast Lane Project. </span><span class="c31 c32">Journal of Open Research Software</span><span class="c36 c31">, 7(1), p.32. DOI: </span><span class="c31 c36"><a class="c30" href="https://www.google.com/url?q=http://doi.org/10.5334/jors.241&amp;sa=D&amp;ust=1605903631402000&amp;usg=AOvVaw0BDca-1E2huSKkcyhFwqOB">http://doi.org/10.5334/jors.241</a></span></p><p class="c8"><span class="c7">5. Wenjin Wang, Bert den Brinker, Sander Stuijk, and Gerard de Haan, &ldquo;Algorithmic Principles of Remote-PPG,&rdquo; http://www.es.ele.tue.nl/~sander/publications/tbme16-algorithmic-rppg.pdf</span></p><hr><p class="c8 c9"><span class="c7"></span></p><p class="c44"><span class="c27">&copy; David Haas, Spencer Mullinix, Hogan Pope</span></p><p class="c8 c9"><span class="c6"></span></p><p class="c8 c9"><span class="c6"></span></p><div class="c20"><p class="c22"><a href="#cmnt_ref1" id="cmnt1">[a]</a><span class="c6">@mullisd1@vt.edu</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref2" id="cmnt2">[b]</a><span class="c6">TODO</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref3" id="cmnt3">[c]</a><span class="c6">@mullisd1@vt.edu update with error table and figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref4" id="cmnt4">[d]</a><span class="c6">@mullisd1@vt.edu update with error table and figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref5" id="cmnt5">[e]</a><span class="c6">@mullisd1@vt.edu update with error table and figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref6" id="cmnt6">[f]</a><span class="c6">@mullisd1@vt.edu update with error table and figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref7" id="cmnt7">[g]</a><span class="c6">@mullisd1@vt.edu update with error table and figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref8" id="cmnt8">[h]</a><span class="c6">@mullisd1@vt.edu update with error table and figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref9" id="cmnt9">[i]</a><span class="c6">Create 2 figures</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref10" id="cmnt10">[j]</a><span class="c6">Decrease image resolution then run experiments and create table</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref11" id="cmnt11">[k]</a><span class="c6">need to update</span></p></div><div class="c20"><p class="c22"><a href="#cmnt_ref12" id="cmnt12">[l]</a><span class="c6">update for dlib</span></p></div></div></body></html>