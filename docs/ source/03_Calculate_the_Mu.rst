3) We will calculate the neutral mutation rate of every especies
-----------------------------------------------------------------

We will use this table from the previous step. We will provide the name of the species for every one. 
then we will divide the  distance_to_Yinptero_Yango_split value between 66,809,683 years
wich is the time calculated in Morales et al., 2026 for that split:

.. code-block::

  ============================================================
  BRANCH LENGTH FROM YINPTERO/YANGO SPLIT TO EACH BAT
  ============================================================
  
      species  distance_to_Yinptero_Yango_split
    HLantPal2                           0.26531
   HLartInt1A                           0.30791
   HLartLit1A                           0.30778
    HLaseSto2                           0.16528
   HLbraCav1A                           0.27791
    HLcarPer2                           0.29846
   HLcenSen1A                           0.32288
   HLchoMin1A                           0.29665
   HLcisSea1A                           0.24017
    HLcorMex1                           0.23799
   HLcorTow1A                           0.23777
   HLcraTho2A                           0.19585
    HLcynSph1                           0.15807
  HLdesRot15A                           0.27438
   HLdiaYou3A                           0.27262
   HLdipEca2A                           0.26811
   HLectAlb1A                           0.31291
   HLeonSpe2A                           0.14218
    HLeptFus3                           0.23229
    HLeptNil3                           0.23179
   HLeroBom1A                           0.27979
   HLeumNan1A                           0.22472
    HLfurHor1                           0.22064
   HLgloMut1A                           0.28582
    HLgloSor2                           0.28574
   HLglyDav1A                           0.29646
   HLhipAba1A                           0.16885
    HLhipArm2                           0.16192
   HLhipCaf2A                           0.16875
    HLhipCyc2                           0.18145
   HLhipJon1A                           0.16850
    HLhipLar2                           0.16062
    HLhipSwi1                           0.16128
    HLhypMon1                           0.14707
   HLlasEga1A                           0.27304
   HLlepYer2A                           0.28144
   HLlioSpu1A                           0.28984
   HLlonInu1A                           0.28900
   HLmacMac1A                           0.28995
   HLmacWat1A                           0.26598
    HLmegSpa1                           0.21540
    HLmicMeg1                           0.27918
    HLminAus1                           0.22781
   HLminNat2B                           0.22633
   HLminSch2A                           0.22444
   HLmolAlv2A                           0.22855
    HLmolMol2                           0.23190
   HLmolNig2A                           0.23200
    HLmopCon1                           0.23183
    HLmorMeg1                           0.23270
    HLmyoAui1                           0.23793
    HLmyoCal2                           0.23773
    HLmyoDau2                           0.24059
    HLmyoEvo1                           0.23592
    HLmyoLuc2                           0.23298
    HLmyoMyo6                           0.24103
   HLmyoMys2A                           0.23090
    HLmyoNig1                           0.23791
    HLmyoOcc1                           0.23339
    HLmyoPil1                           0.23053
    HLmyoThy2                           0.23586
    HLmyoVel1                           0.23660
    HLmyoViv2                           0.23676
    HLmyoVol2                           0.22969
   HLmyoYum1A                           0.23659
    HLmysTub2                           0.21428
    HLmyzAur1                           0.22333
    HLnatTum1                           0.16215
    HLnocLep2                           0.26434
    HLnycAvi1                           0.28242
   HLnycThe1A                           0.24820
    HLphyDis3                           0.29751
    HLphyHas1                           0.29456
    HLpipKuh2                           0.28383
   HLpipNat1A                           0.28469
    HLpipPyg1                           0.28262
   HLplaGui1A                           0.30876
    HLpleAur1                           0.22760
    HLrhiAff2                           0.15524
    HLrhiFer5                           0.15086
   HLrhiHip1A                           0.15399
    HLrhiLuc4                           0.16286
    HLrhiLuc5                           0.16401
    HLrhiMic1                           0.12134
    HLrhiMus1                           0.12404
    HLrhiPea1                           0.15222
   HLrhiPum1A                           0.30834
    HLrhiSed2                           0.16552
    HLrhiSin3                           0.15221
    HLrhiTri2                           0.16447
   HLrhyNas2A                           0.21935
    HLrouAeg4                           0.14412
    HLsacBil1                           0.22191
    HLsacLep1                           0.22085
    HLtadBra3                           0.20253
    HLtapMel1                           0.17297
    HLthyTri1                           0.29057
   HLtraCir4A                           0.27912
   HLtriNic1A                           0.29772
   HLtriPer1A                           0.19921
    HLuroBil1                           0.30872
   HLvamThy1A                           0.30954
    HLvesMur1                           0.29544


.. code-block:: python

  from pathlib import Path
  import pandas as pd
  
  
  # ============================================================
  # 1. FILES
  # ============================================================
  
  desktop = Path.home() / "Desktop"
  
  input_file = desktop / "bat_branch_length_from_Yinptero_Yango_split.tsv"
  
  output_file = desktop / "bat_substitution_rates_from_Yinptero_Yango_split.tsv"
  
  
  # ============================================================
  # 2. DIVERGENCE TIME
  #
  # Yinpterochiroptera / Yangochiroptera split:
  # 66,809,683 years
  # ============================================================
  
  split_time_years = 66_809_683
  
  
  # ============================================================
  # 3. READ TABLE
  # ============================================================
  
  df = pd.read_csv(
      input_file,
      sep="\t"
  )
  
  print("Input file:")
  print(input_file)
  
  print("\nNumber of species:")
  print(len(df))
  
  print("\nColumns:")
  print(df.columns.tolist())
  
  
  # ============================================================
  # 4. VERIFY REQUIRED COLUMN
  # ============================================================
  
  distance_column = "distance_to_Yinptero_Yango_split"
  
  if distance_column not in df.columns:
      raise ValueError(
          f"Column '{distance_column}' was not found in the input table."
      )
  
  
  # ============================================================
  # 5. CALCULATE AVERAGE SUBSTITUTION RATE
  #
  # branch length = substitutions/site
  # time          = years
  #
  # rate = substitutions/site/year
  # ============================================================
  
  df["split_time_years"] = split_time_years
  
  df["substitution_rate_per_site_per_year"] = (
      df[distance_column] / split_time_years
  )
  
  
  # ============================================================
  # 6. OPTIONAL:
  #    ALSO EXPRESS RATE AS x10^-9 substitutions/site/year
  #
  # This is easier to read.
  #
  # Example:
  # 3.60 means 3.60 × 10^-9 substitutions/site/year
  # ============================================================
  
  df["substitution_rate_x1e9"] = (
      df["substitution_rate_per_site_per_year"] * 1e9
  )
  
  
  # ============================================================
  # 7. DISPLAY RESULTS
  # ============================================================
  
  pd.set_option("display.max_rows", None)
  pd.set_option("display.precision", 12)
  
  print("\n============================================================")
  print("LINEAGE-SPECIFIC SUBSTITUTION RATES")
  print("============================================================\n")
  
  print(
      df[
          [
              "species",
              distance_column,
              "split_time_years",
              "substitution_rate_per_site_per_year",
              "substitution_rate_x1e9",
          ]
      ].to_string(index=False)
  )
  
  
  # ============================================================
  # 8. VALIDATE MYOTIS DAUBENTONII
  # ============================================================
  
  target = df[df["species"] == "HLmyoDau2"]
  
  print("\n============================================================")
  print("VALIDATION: Myotis daubentonii")
  print("============================================================\n")
  
  print(
      target[
          [
              "species",
              distance_column,
              "substitution_rate_per_site_per_year",
              "substitution_rate_x1e9",
          ]
      ].to_string(index=False)
  )
  
  
  # ============================================================
  # 9. SAVE RESULTS
  # ============================================================
  
  df.to_csv(
      output_file,
      sep="\t",
      index=False,
      float_format="%.12g"
  )
  
  print("\n============================================================")
  print("OUTPUT SAVED")
  print("============================================================")
  
  print(output_file)



.. code-block::

  ============================================================
  LINEAGE-SPECIFIC SUBSTITUTION RATES
  ============================================================
  
      species  distance_to_Yinptero_Yango_split  split_time_years  substitution_rate_per_site_per_year  substitution_rate_x1e9
    HLantPal2                           0.26531          66809683                       0.000000003971          3.971130951183
   HLartInt1A                           0.30791          66809683                       0.000000004609          4.608763074059
   HLartLit1A                           0.30778          66809683                       0.000000004607          4.606817248332
    HLaseSto2                           0.16528          66809683                       0.000000002474          2.473892893639
   HLbraCav1A                           0.27791          66809683                       0.000000004160          4.159726367808
    HLcarPer2                           0.29846          66809683                       0.000000004467          4.467316511590
   HLcenSen1A                           0.32288          66809683                       0.000000004833          4.832832390478
   HLchoMin1A                           0.29665          66809683                       0.000000004440          4.440224630313
   HLcisSea1A                           0.24017          66809683                       0.000000003595          3.594838191344
    HLcorMex1                           0.23799          66809683                       0.000000003562          3.562208190690
   HLcorTow1A                           0.23777          66809683                       0.000000003559          3.558915254844
   HLcraTho2A                           0.19585          66809683                       0.000000002931          2.931461297309
    HLcynSph1                           0.15807          66809683                       0.000000002366          2.365974405237
  HLdesRot15A                           0.27438          66809683                       0.000000004107          4.106889715373
   HLdiaYou3A                           0.27262          66809683                       0.000000004081          4.080546228606
   HLdipEca2A                           0.26811          66809683                       0.000000004013          4.013041043766
   HLectAlb1A                           0.31291          66809683                       0.000000004684          4.683602525101
   HLeonSpe2A                           0.14218          66809683                       0.000000002128          2.128134629826
    HLeptFus3                           0.23229          66809683                       0.000000003477          3.476891216502
    HLeptNil3                           0.23179          66809683                       0.000000003469          3.469407271398
   HLeroBom1A                           0.27979          66809683                       0.000000004188          4.187866001400
   HLeumNan1A                           0.22472          66809683                       0.000000003364          3.363584287625
    HLfurHor1                           0.22064          66809683                       0.000000003303          3.302515295575
   HLgloMut1A                           0.28582          66809683                       0.000000004278          4.278122379356
    HLgloSor2                           0.28574          66809683                       0.000000004277          4.276924948140
   HLglyDav1A                           0.29646          66809683                       0.000000004437          4.437380731173
   HLhipAba1A                           0.16885          66809683                       0.000000002527          2.527328261683
    HLhipArm2                           0.16192          66809683                       0.000000002424          2.423600782539
   HLhipCaf2A                           0.16875          66809683                       0.000000002526          2.525831472662
    HLhipCyc2                           0.18145          66809683                       0.000000002716          2.715923678309
   HLhipJon1A                           0.16850          66809683                       0.000000002522          2.522089500110
    HLhipLar2                           0.16062          66809683                       0.000000002404          2.404142525268
    HLhipSwi1                           0.16128          66809683                       0.000000002414          2.414021332806
    HLhypMon1                           0.14707          66809683                       0.000000002201          2.201327612945
   HLlasEga1A                           0.27304          66809683                       0.000000004087          4.086832742493
   HLlepYer2A                           0.28144          66809683                       0.000000004213          4.212563020244
   HLlioSpu1A                           0.28984          66809683                       0.000000004338          4.338293297994
   HLlonInu1A                           0.28900          66809683                       0.000000004326          4.325720270219
   HLmacMac1A                           0.28995          66809683                       0.000000004340          4.339939765917
   HLmacWat1A                           0.26598          66809683                       0.000000003981          3.981159437622
    HLmegSpa1                           0.21540          66809683                       0.000000003224          3.224083550883
    HLmicMeg1                           0.27918          66809683                       0.000000004179          4.178735588373
    HLminAus1                           0.22781          66809683                       0.000000003410          3.409835068369
   HLminNat2B                           0.22633          66809683                       0.000000003388          3.387682590860
   HLminSch2A                           0.22444          66809683                       0.000000003359          3.359393278367
   HLmolAlv2A                           0.22855          66809683                       0.000000003421          3.420911307123
    HLmolMol2                           0.23190          66809683                       0.000000003471          3.471053739321
   HLmolNig2A                           0.23200          66809683                       0.000000003473          3.472550528342
    HLmopCon1                           0.23183          66809683                       0.000000003470          3.470005987006
    HLmorMeg1                           0.23270          66809683                       0.000000003483          3.483028051488
    HLmyoAui1                           0.23793          66809683                       0.000000003561          3.561310117277
    HLmyoCal2                           0.23773          66809683                       0.000000003558          3.558316539236
    HLmyoDau2                           0.24059          66809683                       0.000000003601          3.601124705232
    HLmyoEvo1                           0.23592          66809683                       0.000000003531          3.531224657959
    HLmyoLuc2                           0.23298          66809683                       0.000000003487          3.487219060746
    HLmyoMyo6                           0.24103          66809683                       0.000000003608          3.607710576923
   HLmyoMys2A                           0.23090          66809683                       0.000000003456          3.456085849113
    HLmyoNig1                           0.23791          66809683                       0.000000003561          3.561010759473
    HLmyoOcc1                           0.23339          66809683                       0.000000003493          3.493355895731
    HLmyoPil1                           0.23053          66809683                       0.000000003451          3.450547729736
    HLmyoThy2                           0.23586          66809683                       0.000000003530          3.530326584546
    HLmyoVel1                           0.23660          66809683                       0.000000003541          3.541402823300
    HLmyoViv2                           0.23676          66809683                       0.000000003544          3.543797685734
    HLmyoVol2                           0.22969          66809683                       0.000000003438          3.437974701960
   HLmyoYum1A                           0.23659          66809683                       0.000000003541          3.541253144398
    HLmysTub2                           0.21428          66809683                       0.000000003207          3.207319513850
    HLmyzAur1                           0.22333          66809683                       0.000000003343          3.342778920235
    HLnatTum1                           0.16215          66809683                       0.000000002427          2.427043397287
    HLnocLep2                           0.26434          66809683                       0.000000003957          3.956612097681
    HLnycAvi1                           0.28242          66809683                       0.000000004227          4.227231552648
   HLnycThe1A                           0.24820          66809683                       0.000000003715          3.715030349717
    HLphyDis3                           0.29751          66809683                       0.000000004453          4.453097015892
    HLphyHas1                           0.29456          66809683                       0.000000004409          4.408941739777
    HLpipKuh2                           0.28383          66809683                       0.000000004248          4.248336277842
   HLpipNat1A                           0.28469          66809683                       0.000000004261          4.261208663421
    HLpipPyg1                           0.28262          66809683                       0.000000004230          4.230225130690
   HLplaGui1A                           0.30876          66809683                       0.000000004621          4.621485780736
    HLpleAur1                           0.22760          66809683                       0.000000003407          3.406691811425
    HLrhiAff2                           0.15524          66809683                       0.000000002324          2.323615275947
    HLrhiFer5                           0.15086          66809683                       0.000000002258          2.258055916835
   HLrhiHip1A                           0.15399          66809683                       0.000000002305          2.304905413187
    HLrhiLuc4                           0.16286          66809683                       0.000000002438          2.437670599335
    HLrhiLuc5                           0.16401          66809683                       0.000000002455          2.454883673075
    HLrhiMic1                           0.12134          66809683                       0.000000001816          1.816203797884
    HLrhiMus1                           0.12404          66809683                       0.000000001857          1.856617101446
    HLrhiPea1                           0.15222          66809683                       0.000000002278          2.278412247518
   HLrhiPum1A                           0.30834          66809683                       0.000000004615          4.615199266849
    HLrhiSed2                           0.16552          66809683                       0.000000002477          2.477485187289
    HLrhiSin3                           0.15221          66809683                       0.000000002278          2.278262568616
    HLrhiTri2                           0.16447          66809683                       0.000000002462          2.461768902571
   HLrhyNas2A                           0.21935          66809683                       0.000000003283          3.283206717206
    HLrouAeg4                           0.14412          66809683                       0.000000002157          2.157172336830
    HLsacBil1                           0.22191          66809683                       0.000000003322          3.321524516139
    HLsacLep1                           0.22085          66809683                       0.000000003306          3.305658552519
    HLtadBra3                           0.20253          66809683                       0.000000003031          3.031446803901
    HLtapMel1                           0.17297          66809683                       0.000000002589          2.588995969342
    HLthyTri1                           0.29057          66809683                       0.000000004349          4.349219857846
   HLtraCir4A                           0.27912          66809683                       0.000000004178          4.177837514960
   HLtriNic1A                           0.29772          66809683                       0.000000004456          4.456240272836
   HLtriPer1A                           0.19921          66809683                       0.000000002982          2.981753408409
    HLuroBil1                           0.30872          66809683                       0.000000004621          4.620887065128
   HLvamThy1A                           0.30954          66809683                       0.000000004633          4.633160735099
    HLvesMur1                           0.29544          66809683                       0.000000004422          4.422113483161
  
  ============================================================
  VALIDATION: Myotis daubentonii
  ============================================================
  
    species  distance_to_Yinptero_Yango_split  substitution_rate_per_site_per_year  substitution_rate_x1e9
  HLmyoDau2                           0.24059                       0.000000003601          3.601124705232


Now we must create a dictionary:

.. code-block:: 

  species_names = {
      "HLantPal2":   "Antrozous pallidus",
      "HLartInt1A":  "Artibeus intermedius",
      "HLartLit1A":  "Artibeus lituratus",
      "HLaseSto2":   "Aselliscus stoliczkanus",
      "HLbraCav1A":  "Brachyphylla cavernarum",
      "HLcarPer2":   "Carollia perspicillata",
      "HLcenSen1A":  "Centurio senex",
      "HLchoMin1A":  "Choeroniscus minor",
      "HLcisSea1A":  "Cistugo seabrae",
      "HLcorMex1":   "Corynorhinus mexicanus",
      "HLcorTow1A":  "Corynorhinus townsendii",
      "HLcraTho2A":  "Craseonycteris thonglongyai",
      "HLcynSph1":   "Cynopterus sphinx",
      "HLdesRot15A": "Desmodus rotundus",
      "HLdiaYou3A":  "Diaemus youngii",
      "HLdipEca2A":  "Diphylla ecaudata",
      "HLectAlb1A":  "Ectophylla alba",
      "HLeonSpe2A":  "Eonycteris spelaea",
      "HLeptFus3":   "Eptesicus fuscus",
      "HLeptNil3":   "Eptesicus nilssonii",
      "HLeroBom1A":  "Erophylla bombifrons",
      "HLeumNan1A":  "Eumops nanus",
      "HLfurHor1":   "Furipterus horrens",
      "HLgloMut1A":  "Glossophaga mutica",
      "HLgloSor2":   "Glossophaga soricina",
      "HLglyDav1A":  "Glyphonycteris daviesi",
      "HLhipAba1A":  "Hipposideros abae",
      "HLhipArm2":   "Hipposideros armiger",
      "HLhipCaf2A":  "Hipposideros caffer",
      "HLhipCyc2":   "Doryrhina cyclops",
      "HLhipJon1A":  "Hipposideros jonesi",
      "HLhipLar2":   "Hipposideros larvatus",
      "HLhipSwi1":   "Hipposideros swinhoii",
      "HLhypMon1":   "Hypsignathus monstrosus",
      "HLlasEga1A":  "Dasypterus ega",
      "HLlepYer2A":  "Leptonycteris yerbabuenae",
      "HLlioSpu1A":  "Lionycteris spurrelli",
      "HLlonInu1A":  "Lonchorhina inusitata",
      "HLmacMac1A":  "Macrophyllum macrophyllum",
      "HLmacWat1A":  "Macrotus waterhousii",
      "HLmegSpa1":   "Megaderma spasma",
      "HLmicMeg1":   "Micronycteris megalotis",
      "HLminAus1":   "Miniopterus australis",
      "HLminNat2B":  "Miniopterus natalensis",
      "HLminSch2A":  "Miniopterus schreibersii",
      "HLmolAlv2A":  "Molossus alvarezi",
      "HLmolMol2":   "Molossus molossus",
      "HLmolNig2A":  "Molossus nigricans",
      "HLmopCon1":   "Mops condylurus",
      "HLmorMeg1":   "Mormoops megalophylla",
      "HLmyoAui1":   "Myotis auriculus",
      "HLmyoCal2":   "Myotis californicus",
      "HLmyoDau2":   "Myotis daubentonii",
      "HLmyoEvo1":   "Myotis evotis",
      "HLmyoLuc2":   "Myotis lucifugus",
      "HLmyoMyo6":   "Myotis myotis",
      "HLmyoMys2A":  "Myotis mystacinus",
      "HLmyoNig1":   "Myotis nigricans",
      "HLmyoOcc1":   "Myotis occultus",
      "HLmyoPil1":   "Myotis ricketti",
      "HLmyoThy2":   "Myotis thysanodes",
      "HLmyoVel1":   "Myotis velifer",
      "HLmyoViv2":   "Myotis vivesi",
      "HLmyoVol2":   "Myotis volans",
      "HLmyoYum1A":  "Myotis yumanensis",
      "HLmysTub2":   "Mystacina tuberculata",
      "HLmyzAur1":   "Myzopoda aurita",
      "HLnatTum1":   "Natalus tumidirostris",
      "HLnocLep2":   "Noctilio leporinus",
      "HLnycAvi1":   "Nyctalus aviator",
      "HLnycThe1A":  "Nycteris thebaica",
      "HLphyDis3":   "Phyllostomus discolor",
      "HLphyHas1":   "Phyllostomus hastatus",
      "HLpipKuh2":   "Pipistrellus kuhlii",
      "HLpipNat1A":  "Pipistrellus nathusii",
      "HLpipPyg1":   "Pipistrellus pygmaeus",
      "HLplaGui1A":  "Platyrrhinus guianensis",
      "HLpleAur1":   "Plecotus auritus",
      "HLrhiAff2":   "Rhinolophus affinis",
      "HLrhiFer5":   "Rhinolophus ferrumequinum",
      "HLrhiHip1A":  "Rhinolophus hipposideros",
      "HLrhiLuc4":   "Rhinolophus perniger lanosus",
      "HLrhiLuc5":   "Rhinolophus foetidus",
      "HLrhiMic1":   "Rhinopoma microphyllum",
      "HLrhiMus1":   "Rhinopoma muscatellum",
      "HLrhiPea1":   "Rhinolophus pearsonii",
      "HLrhiPum1A":  "Rhinophylla pumilio",
      "HLrhiSed2":   "Rhinolophus yonghoiseni",
      "HLrhiSin3":   "Rhinolophus sinicus",
      "HLrhiTri2":   "Rhinolophus trifoliatus",
      "HLrhyNas2A":  "Rhynchonycteris naso",
      "HLrouAeg4":   "Rousettus aegyptiacus",
      "HLsacBil1":   "Saccopteryx bilineata",
      "HLsacLep1":   "Saccopteryx leptura",
      "HLtadBra3":   "Tadarida brasiliensis",
      "HLtapMel1":   "Taphozous melanopogon",
      "HLthyTri1":   "Thyroptera tricolor",
      "HLtraCir4A":  "Trachops cirrhosus",
      "HLtriNic1A":  "Trinycteris nicefori",
      "HLtriPer1A":  "Triaenops persicus",
      "HLuroBil1":   "Uroderma convexum",
      "HLvamThy1A":  "Vampyressa thyone",
      "HLvesMur1":   "Vespertilio murinus",
  }


  # Add scientific names
  df["scientific_name"] = df["species"].map(species_names)
  
  # Verify that all 103 species received a name
  missing = df.loc[df["scientific_name"].isna(), "species"].tolist()
  
  if missing:
      raise ValueError(
          "Missing scientific names for:\n" +
          "\n".join(missing)
      )
  
  print("Species with scientific names:", df["scientific_name"].notna().sum())
  
  # Put scientific_name immediately after the tree-tip code
  cols = ["species", "scientific_name"] + [
      c for c in df.columns
      if c not in ["species", "scientific_name"]
  ]
  
  df = df[cols]

  output_file = (
      desktop /
      "bat_neutral_substitution_rates_with_species_names.tsv"
  )
  
  df.to_csv(
      output_file,
      sep="\t",
      index=False,
      float_format="%.12g"
  )
  
  print(output_file)


.. csv-table:: _static/Neutral substitution rates for bat species
   :file: bat_neutral_substitution_rates_with_species_names.tsv
   :delim: tab
   :header-rows: 1
   :widths: auto
