2) We must find the split between Yinpterochirotera y Yangochiroptera
----------------------------------------------------------------------

.. code-block:: python

  from Bio import Phylo
  from pathlib import Path
  import pandas as pd
  
  desktop = Path.home() / "Desktop"
  
  tree_file = desktop / "Myotis_cactus_neutral_ML_rooted.tre"
  
  tree = Phylo.read(tree_file, "newick")
  
  # ============================================================
  # 1. Find the SPLIT YINPTEROCHIROPTERA / YANGOCHIROPTERA
  # ============================================================
  
  bat_root = tree.common_ancestor(
      "HLcynSph1",   # Cynopterus sphinx = Yinpterochiroptera
      "HLmyoDau2"    # Myotis daubentonii = Yangochiroptera
  )
  
  print("Número de tips descendientes del nodo basal de Chiroptera:",
        len(bat_root.get_terminals()))
