2) We must find the split between Yinpterochirotera y Yangochiroptera
----------------------------------------------------------------------

.. code-block:: python

  from Bio import Phylo
  from pathlib import Path
  import pandas as pd
  
  
  # ============================================================
  # 1. FILES
  # ============================================================
  
  desktop = Path.home() / "Desktop"
  
  tree_file = desktop / "Myotis_cactus_neutral_ML_rooted.tre"
  
  output_file = desktop / "bat_branch_length_from_Yinptero_Yango_split.tsv"
  
  
  # ============================================================
  # 2. read rooted tree
  # ============================================================
  
  tree = Phylo.read(tree_file, "newick")
  
  print("Tree loaded:")
  print(tree_file)
  
  print("\nTotal number of tips in tree:")
  print(len(tree.get_terminals()))
  
  
  # ============================================================
  # 3. DEFINE ONE REPRESENTATIVE FROM EACH MAJOR BAT CLADE
  #
  # Yinpterochiroptera:
  #   HLcynSph1 = Cynopterus sphinx
  #
  # Yangochiroptera:
  #   HLmyoDau2 = Myotis daubentonii
  #
  # Their MRCA should therefore correspond to the basal
  # Yinpterochiroptera / Yangochiroptera split.
  # ============================================================
  
  yinptero_example = "HLcynSph1"
  yango_example = "HLmyoDau2"
  
  bat_root = tree.common_ancestor(
      yinptero_example,
      yango_example
  )
  
  
  # ============================================================
  # 4. VERIFY THAT THIS NODE CONTAINS ONLY CHIROPTERA
  # ============================================================
  
  bat_tips = bat_root.get_terminals()
  bat_names = {tip.name for tip in bat_tips}
  
  print("\nNumber of tips descending from the basal Chiroptera node:")
  print(len(bat_tips))
  
  
  # Known non-bat mammals in this tree
  outgroups = {
      "hg38",
      "mm39",
      "HLeleMax2",
      "HLsorFum1",
      "HLbosTau10",
      "HLneoVis2",
      "HLmanPen4",
      "HLequAsi3",
  }
  
  outgroups_inside_bat_clade = bat_names.intersection(outgroups)
  
  if outgroups_inside_bat_clade:
      raise RuntimeError(
          "ERROR: non-bat taxa were found inside the Chiroptera node: "
          f"{sorted(outgroups_inside_bat_clade)}"
      )
  
  print("No non-bat outgroups occur inside this node.")
  
  if len(bat_tips) != 103:
      print(
          "\nWARNING: expected 103 bat tips, "
          f"but found {len(bat_tips)}."
      )
  else:
      print("The node contains the expected 103 bat tips.")
  
  
  # ============================================================
  # 5. CALCULATE CUMULATIVE BRANCH LENGTH
  #    FROM THE YINPTERO/YANGO SPLIT TO EACH BAT TIP
  # ============================================================
  
  rows = []
  
  for tip in bat_tips:
  
      distance = tree.distance(bat_root, tip)
  
      rows.append({
          "species": tip.name,
          "distance_to_Yinptero_Yango_split": distance
      })
  
  
  # ============================================================
  # 6. CREATE DATAFRAME
  # ============================================================
  
  df = pd.DataFrame(rows)
  
  df = df.sort_values("species").reset_index(drop=True)
  
  pd.set_option("display.max_rows", None)
  pd.set_option("display.precision", 8)
  
  
  # ============================================================
  # 7. DISPLAY ALL 103 BAT SPECIES
  # ============================================================
  
  print("\n============================================================")
  print("BRANCH LENGTH FROM YINPTERO/YANGO SPLIT TO EACH BAT")
  print("============================================================\n")
  
  print(df.to_string(index=False))
  
  
  # ============================================================
  # 8. VALIDATION USING MYOTIS DAUBENTONII
  #
  # Expected value from the manual calculation:
  # approximately 0.24072
  # ============================================================
  
  target_name = "HLmyoDau2"
  
  target_tip = next(
      tip for tip in bat_tips
      if tip.name == target_name
  )
  
  # Path from basal Chiroptera node to Myotis daubentonii
  path = bat_root.get_path(target_tip)
  
  branch_lengths = [
      clade.branch_length or 0.0
      for clade in path
  ]
  
  manual_sum = sum(branch_lengths)
  
  automatic_distance = tree.distance(
      bat_root,
      target_tip
  )
  
  print("\n============================================================")
  print("VALIDATION: Myotis daubentonii")
  print("============================================================")
  
  print("\nTip:")
  print(target_name)
  
  print("\nIndividual branch lengths:")
  
  for i, length in enumerate(branch_lengths, start=1):
      print(f"Branch {i:2d}: {length:.5f}")
  
  print("\nSum written as an equation:")
  
  print(
      " + ".join(
          f"{length:.5f}"
          for length in branch_lengths
      )
  )
  
  print("\nManual sum from branch lengths:")
  print(f"{manual_sum:.5f}")
  
  print("\nBiopython tree.distance result:")
  print(f"{automatic_distance:.5f}")
  
  if abs(manual_sum - automatic_distance) < 1e-12:
      print("\nOK: manual branch sum and tree.distance are identical.")
  else:
      print("\nWARNING: the two calculations do not agree.")
  
  
  # ============================================================
  # 9. SHOW THE MYOTIS DAUBENTONII ROW
  # ============================================================
  
  print("\n============================================================")
  print("Myotis daubentonii result in dataframe")
  print("============================================================\n")
  
  print(
      df[df["species"] == target_name].to_string(index=False)
  )
  
  
  # ============================================================
  # 10. SAVE RESULTS
  # ============================================================
  
  df.to_csv(
      output_file,
      sep="\t",
      index=False,
      float_format="%.8f"
  )
  
  print("\n============================================================")
  print("OUTPUT SAVED")
  print("============================================================")
  
  print(output_file)


The result should be this:

.. code-block:: 

  Tree loaded:
  /Users/manuelhoyos/Desktop/Myotis_cactus_neutral_ML_rooted.tre
  
  Total number of tips in tree:
  111
  
  Number of tips descending from the basal Chiroptera node:
  103
  No non-bat outgroups occur inside this node.
  The node contains the expected 103 bat tips.
  
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
  
  ============================================================
  VALIDATION: Myotis daubentonii
  ============================================================
  
  Tip:
  HLmyoDau2
  
  Individual branch lengths:
  Branch  1: 0.06323
  Branch  2: 0.00274
  Branch  3: 0.02114
  Branch  4: 0.00783
  Branch  5: 0.00936
  Branch  6: 0.02226
  Branch  7: 0.06240
  Branch  8: 0.02814
  Branch  9: 0.00110
  Branch 10: 0.01001
  Branch 11: 0.01238
  
  Sum written as an equation:
  0.06323 + 0.00274 + 0.02114 + 0.00783 + 0.00936 + 0.02226 + 0.06240 + 0.02814 + 0.00110 + 0.01001 + 0.01238
  
  Manual sum from branch lengths:
  0.24059
  
  Biopython tree.distance result:
  0.24059
  
  OK: manual branch sum and tree.distance are identical.
  
  ============================================================
  Myotis daubentonii result in dataframe
  ============================================================
  
    species  distance_to_Yinptero_Yango_split
  HLmyoDau2                           0.24059
  
  ============================================================
  OUTPUT SAVED
  ============================================================
  /Users/manuelhoyos/Desktop/bat_branch_length_from_Yinptero_Yango_split.tsv
  
  
  
