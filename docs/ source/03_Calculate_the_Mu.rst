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
