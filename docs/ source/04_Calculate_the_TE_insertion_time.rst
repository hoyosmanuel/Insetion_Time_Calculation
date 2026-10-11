4) The first thing is to polish the RepeatMasker output and then add the time
------------------------------------------------------------------------------

4.1 polish the RepeatMasker output
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

  cat > clean_all_repeatmasker.py <<'PY'
  from pathlib import Path
  import csv
  import sys
  import time
  
  
  # ============================================================
  # CONFIGURATION
  # ============================================================
  
  INPUT_DIR = Path(".")
  
  # False = if a .clean.tsv already exists, do not redo it
  # True  = overwrite existing .clean.tsv files
  OVERWRITE = False
  
  
  # ============================================================
  # CLEAN HEADER
  # ============================================================
  
  HEADER = [
      "SW_score",
      "perc_div",
      "perc_del",
      "perc_ins",
      "query_sequence",
      "query_begin",
      "query_end",
      "query_left",
      "strand",
      "matching_repeat",
      "repeat_class_family",
      "repeat_position_1",
      "repeat_position_2",
      "repeat_position_3",
      "ID",
      "overlap_flag",
  ]
  
  
  # ============================================================
  # CLEAN ONE REPEATMASKER FILE
  # ============================================================
  
  def clean_repeatmasker_file(input_file, output_file):
  
      rows_written = 0
      rows_without_id = 0
      rows_with_overlap = 0
  
      # Write first to a temporary file.
      # If something fails, we do not leave a fake "finished" TSV.
      temp_file = Path(str(output_file) + ".tmp")
  
      try:
  
          with input_file.open(
              "r",
              encoding="utf-8",
              errors="replace"
          ) as fin, temp_file.open(
              "w",
              encoding="utf-8",
              newline=""
          ) as fout:
  
              writer = csv.writer(
                  fout,
                  delimiter="\t",
                  lineterminator="\n"
              )
  
              # ------------------------------------------------
              # One single clean header
              # ------------------------------------------------
  
              writer.writerow(HEADER)
  
              # ------------------------------------------------
              # Read original RepeatMasker file line by line
              # ------------------------------------------------
  
              for line_number, line in enumerate(fin, start=1):
  
                  fields = line.split()
  
                  # Skip blank lines
                  if not fields:
                      continue
  
                  # --------------------------------------------
                  # Detect actual RepeatMasker data rows.
                  #
                  # Original header lines fail this test and
                  # therefore disappear automatically.
                  # --------------------------------------------
  
                  try:
                      float(fields[0])   # SW score
                      float(fields[1])   # perc.div
                      float(fields[2])   # perc.del
                      float(fields[3])   # perc.ins
  
                  except (ValueError, IndexError):
                      continue
  
                  # --------------------------------------------
                  # Optional RepeatMasker overlap marker "*"
                  # --------------------------------------------
  
                  overlap_flag = ""
  
                  if fields[-1] == "*":
                      overlap_flag = "*"
                      fields = fields[:-1]
                      rows_with_overlap += 1
  
                  # --------------------------------------------
                  # RepeatMasker rows encountered in our data:
                  #
                  # 14 fields = ID absent
                  # 15 fields = ID present
                  # --------------------------------------------
  
                  if len(fields) == 14:
  
                      core = fields
                      repeat_id = ""
                      rows_without_id += 1
  
                  elif len(fields) == 15:
  
                      core = fields[:14]
                      repeat_id = fields[14]
  
                  else:
  
                      raise ValueError(
                          "\nUnexpected RepeatMasker structure\n"
                          f"File: {input_file.name}\n"
                          f"Line: {line_number}\n"
                          f"Fields after removing optional '*': "
                          f"{len(fields)}\n"
                          f"Content:\n{line}"
                      )
  
                  # --------------------------------------------
                  # Write clean tab-separated row
                  #
                  # NO calculations are performed here.
                  # perc_div remains exactly as RepeatMasker
                  # reported it.
                  # --------------------------------------------
  
                  new_row = (
                      core
                      + [repeat_id]
                      + [overlap_flag]
                  )
  
                  if len(new_row) != len(HEADER):
                      raise ValueError(
                          f"Internal column-count error in "
                          f"{input_file.name}, line {line_number}"
                      )
  
                  writer.writerow(new_row)
  
                  rows_written += 1
  
          # Only rename when the entire file finished correctly
          temp_file.replace(output_file)
  
          return {
              "rows_written": rows_written,
              "rows_without_id": rows_without_id,
              "rows_with_overlap": rows_with_overlap,
          }
  
      except Exception:
  
          # Delete incomplete temporary output
          if temp_file.exists():
              temp_file.unlink()
  
          raise
  
  
  # ============================================================
  # MAIN
  # ============================================================
  
  input_files = sorted(INPUT_DIR.glob("*.fa.out"))
  
  if not input_files:
      print("No *.fa.out files found in:")
      print(INPUT_DIR.resolve())
      sys.exit(1)
  
  
  print()
  print("RepeatMasker files found:", len(input_files))
  print("Directory:", INPUT_DIR.resolve())
  print()
  print("IMPORTANT:")
  print("This step only cleans the tables.")
  print("No divergence or insertion-age calculations are performed.")
  print()
  
  
  summary = []
  failures = []
  
  start_all = time.time()
  
  
  for index, input_file in enumerate(input_files, start=1):
  
      # Example:
      #
      # mAntPal2.1.pri.fa.out
      # ->
      # mAntPal2.1.pri.fa.clean.tsv
  
      output_name = (
          input_file.name.removesuffix(".out")
          + ".clean.tsv"
      )
  
      output_file = input_file.parent / output_name
  
      print(
          f"[{index}/{len(input_files)}] "
          f"{input_file.name}"
      )
  
      # --------------------------------------------------------
      # Do not redo existing files unless OVERWRITE=True
      # --------------------------------------------------------
  
      if output_file.exists() and not OVERWRITE:
  
          print(
              f"    SKIPPED: {output_file.name} already exists"
          )
  
          summary.append({
              "input_file": input_file.name,
              "output_file": output_file.name,
              "status": "SKIPPED",
              "rows_written": "",
              "rows_without_id": "",
              "rows_with_overlap": "",
          })
  
          continue
  
      start_file = time.time()
  
      try:
  
          stats = clean_repeatmasker_file(
              input_file,
              output_file
          )
  
          elapsed = time.time() - start_file
  
          print(
              f"    OK: {stats['rows_written']:,} rows"
          )
  
          print(
              f"    rows without ID: "
              f"{stats['rows_without_id']:,}"
          )
  
          print(
              f"    rows with '*': "
              f"{stats['rows_with_overlap']:,}"
          )
  
          print(
              f"    output: {output_file.name}"
          )
  
          print(
              f"    time: {elapsed / 60:.2f} min"
          )
  
          summary.append({
              "input_file": input_file.name,
              "output_file": output_file.name,
              "status": "OK",
              "rows_written": stats["rows_written"],
              "rows_without_id": stats["rows_without_id"],
              "rows_with_overlap": stats["rows_with_overlap"],
          })
  
      except Exception as error:
  
          print("    ERROR")
          print(str(error))
  
          failures.append(
              (input_file.name, str(error))
          )
  
          summary.append({
              "input_file": input_file.name,
              "output_file": output_file.name,
              "status": "ERROR",
              "rows_written": "",
              "rows_without_id": "",
              "rows_with_overlap": "",
          })
  
      print()
  
  
  # ============================================================
  # WRITE SUMMARY TABLE
  # ============================================================
  
  summary_file = INPUT_DIR / "repeatmasker_cleaning_summary.tsv"
  
  with summary_file.open(
      "w",
      encoding="utf-8",
      newline=""
  ) as fout:
  
      fieldnames = [
          "input_file",
          "output_file",
          "status",
          "rows_written",
          "rows_without_id",
          "rows_with_overlap",
      ]
  
      writer = csv.DictWriter(
          fout,
          fieldnames=fieldnames,
          delimiter="\t",
          lineterminator="\n"
      )
  
      writer.writeheader()
      writer.writerows(summary)
  
  
  # ============================================================
  # FINAL REPORT
  # ============================================================
  
  elapsed_all = time.time() - start_all
  
  successful = sum(
      row["status"] == "OK"
      for row in summary
  )
  
  skipped = sum(
      row["status"] == "SKIPPED"
      for row in summary
  )
  
  print("=" * 70)
  print("FINISHED")
  print("=" * 70)
  
  print("Input files:", len(input_files))
  print("Successfully cleaned:", successful)
  print("Skipped:", skipped)
  print("Failed:", len(failures))
  print(f"Total time: {elapsed_all / 60:.2f} min")
  print("Summary:", summary_file.resolve())
  
  
  if failures:
  
      print()
      print("FILES WITH ERRORS:")
  
      for filename, error in failures:
          print()
          print(filename)
          print(error)
  
      sys.exit(1)
  
  else:
  
      print()
      print("All requested files completed without errors.")
  PY

.. code-block:: bash

  python3 clean_all_repeatmasker.py



4.2 Add the insertion time to each TE
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  
.. code-block:: bash
  
  cat > add_insertion_ages.py <<'PY'
  from pathlib import Path
  import csv
  import sys
  import time
  
  
  # ============================================================
  # CONFIGURATION
  # ============================================================
  
  DATA_DIR = Path(".")
  
  RATES_FILE = (
      Path.home()
      / "Desktop"
      / "bat_neutral_substitution_rates_with_species_names.tsv"
  )
  
  OVERWRITE = False
  
  
  # ============================================================
  # EXPECTED CLEAN TSV HEADER
  # ============================================================
  
  EXPECTED_HEADER = [
      "SW_score",
      "perc_div",
      "perc_del",
      "perc_ins",
      "query_sequence",
      "query_begin",
      "query_end",
      "query_left",
      "strand",
      "matching_repeat",
      "repeat_class_family",
      "repeat_position_1",
      "repeat_position_2",
      "repeat_position_3",
      "ID",
      "overlap_flag",
  ]
  
  
  # ============================================================
  # ASSEMBLY -> TREE CODE
  #
  # None = species identified, but no species-specific mu
  #        available in the current rate table.
  # ============================================================
  
  assembly_to_tree_code = {
  
      "mAntPal2.1.pri":   "HLantPal2",
      "mArtIn1.1.hap1":   "HLartInt1A",
      "mArtLit1.1.hap1":  "HLartLit1A",
      "mAseSto1.2.pri":   "HLaseSto2",
      "mBraCav1.1.hap1":  "HLbraCav1A",
      "mCarPer1.2.pri":    "HLcarPer2",
      "mCenSen1.1.hap1":  "HLcenSen1A",
      "mChoMin1.1.hap1":   "HLchoMin1A",
      "mCisSea1.1.hap1":   "HLcisSea1A",
      "mCorMex1.1.pri":    "HLcorMex1",
      "mCorTow1.1.hap1":   "HLcorTow1A",
      "mCraTho2.1.hap1":   "HLcraTho2A",
      "mCynSph1.1.pri":    "HLcynSph1",
      "mDesRot1.15.hap1":  "HLdesRot15A",
      "mDiaYou1.3.hap1":   "HLdiaYou3A",
      "mDipEca1.2.hap1":   "HLdipEca2A",
      "mEctAlb1.1.hap1":   "HLectAlb1A",
      "mEonSpe1.2.hap1":   "HLeonSpe2A",
      "mEptFus1.3.pri":    "HLeptFus3",
      "mEptNil2.1.pri":    "HLeptNil3",
      "mEroBom1.1.hap1":   "HLeroBom1A",
      "mEumNan1.1.hap1":   "HLeumNan1A",
      "mFurHor1.1.pri":    "HLfurHor1",
      "mGloMut1.1.hap1":   "HLgloMut1A",
      "mGloSor1.2.pri":    "HLgloSor2",
      "mGlyDav1.1.hap1":   "HLglyDav1A",
  
      "mHipAba1.1.hap1":   "HLhipAba1A",
      "mHipArm2.1.pri":    "HLhipArm2",
      "mHipCaf2.1.hap1":   "HLhipCaf2A",
      "mHipJon1.1.hap1":   "HLhipJon1A",
      "mHipLar1.2.pri":    "HLhipLar2",
      "mHipSwi1.1.pri":    "HLhipSwi1",
  
      "mHypMon1.1.pri":    "HLhypMon1",
      "mLepYer2.1.hap1":   "HLlepYer2A",
      "mLioSpu1.1.hap1":   "HLlioSpu1A",
      "mLonInu1.1.hap1":   "HLlonInu1A",
      "mMacMac1.1.hap1":   "HLmacMac1A",
      "mMacWat1.1.hap1":   "HLmacWat1A",
      "mMegSpa1.1.pri":    "HLmegSpa1",
      "mMicMeg1.1.pri":    "HLmicMeg1",
  
      "mMinAus1.1.pri":    "HLminAus1",
      "mMinNat1.1.hap2":   "HLminNat2B",
      "mMinSch1.1.hap1":   "HLminSch2A",
  
      "mMolAlv1.1.hap1":   "HLmolAlv2A",
      "mMolMol1.2.pri":    "HLmolMol2",
      "mMolNig1.2.hap1":   "HLmolNig2A",
      "mMopCon1.1":        "HLmopCon1",
      "mMorMeg1.1.pri":    "HLmorMeg1",
  
      "mMyoAur1.1.pri":    "HLmyoAui1",
      "mMyoCal1.2.pri":    "HLmyoCal2",
      "mMyoDau2.1.pri":    "HLmyoDau2",
      "mMyoEvo1.2.pri":    "HLmyoEvo1",
      "mMyoLuc2.1.pri":    "HLmyoLuc2",
      "mMyoMyo1.6.pri":    "HLmyoMyo6",
      "mMyoMys1.1.hap1":   "HLmyoMys2A",
      "mMyoNig1.1.pri":    "HLmyoNig1",
      "mMyoOcc1.1.pri":    "HLmyoOcc1",
      "mMyoPil1.2.pri":    "HLmyoPil1",
      "mMyoThy1.2.pri":    "HLmyoThy2",
  
      # Myotis tricolor — excluded: no species-specific mu
      "mMyoTri1.1.hap1":   None,
  
      "mMyoVel1.1.pri":    "HLmyoVel1",
      "mMyoViv1.2.pri":    "HLmyoViv2",
      "mMyoVol1.2.pri":    "HLmyoVol2",
      "mMyoYum1.1.hap1":   "HLmyoYum1A",
  
      "mMysTub1.2.pri":    "HLmysTub2",
      "mMyzAur1.1.pri":    "HLmyzAur1",
      "mNatTum1.1.pri":    "HLnatTum1",
      "mNocLep2.1.pri":    "HLnocLep2",
      "mNycAvi1.1.pri":    "HLnycAvi1",
      "mNycThe1.1.hap1":   "HLnycThe1A",
  
      "mPhyDis1.3.pri":    "HLphyDis3",
      "mPhyHas1.1.pri":    "HLphyHas1",
      "mPipKuh1.2.pri":    "HLpipKuh2",
      "mPipNat1.1.hap1":   "HLpipNat1A",
      "mPipPyg1.1.pri":    "HLpipPyg1",
      "mPlaGui1.1.hap1":   "HLplaGui1A",
      "mPleAur1.1.pri":    "HLpleAur1",
  
      "mRhiAff1.2.pri":    "HLrhiAff2",
      "mRhiFer1.5.pri":    "HLrhiFer5",
      "mRhiFoe1.2.pri":    "HLrhiLuc5",
      "mRhiHip1.1.hap1":   "HLrhiHip1A",
      "mRhiMic1.1.pri":    "HLrhiMic1",
      "mRhiMus1.1.pri":    "HLrhiMus1",
      "mRhiPea1.1.pri":    "HLrhiPea1",
      "mRhiPer1.2.pri":    "HLrhiLuc4",
      "mRhiPum1.1.hap1":   "HLrhiPum1A",
      "mRhiSin3.1.pri":    "HLrhiSin3",
      "mRhiTri1.2.pri":    "HLrhiTri2",
      "mRhiYon1.2.pri":    "HLrhiSed2",
  
      "mRhyNas1.2.hap1":   "HLrhyNas2A",
      "mRouAeg1.4.pri":    "HLrouAeg4",
      "mSacBil1.1.pri":    "HLsacBil1",
      "mSacLep1.1.pri":    "HLsacLep1",
      "mTadBra1.3.pri":    "HLtadBra3",
      "mTapMel1.1.pri":    "HLtapMel1",
      "mThyTri1.1.pri":    "HLthyTri1",
      "mTraCir3.1.hap1":   "HLtraCir4A",
      "mTriNic1.1.hap1":   "HLtriNic1A",
      "mTriPer1.1.hap1":   "HLtriPer1A",
  
      "mUroCon1.1.pri":    "HLuroBil1",
  
      "mVamThy1.1.hap1":   "HLvamThy1A",
      "mVesMur1.1.pri":    "HLvesMur1",
  }
  
  
  # ============================================================
  # LOAD RATE TABLE
  # ============================================================
  
  if not RATES_FILE.exists():
      print("ERROR: rate table not found:")
      print(RATES_FILE)
      sys.exit(1)
  
  
  rate_by_code = {}
  scientific_name_by_code = {}
  
  
  with RATES_FILE.open(
      "r",
      encoding="utf-8",
      newline=""
  ) as fin:
  
      reader = csv.DictReader(fin, delimiter="\t")
  
      required_columns = {
          "species",
          "scientific_name",
          "substitution_rate_x1e9",
      }
  
      if not required_columns.issubset(reader.fieldnames or []):
          print("ERROR: rate table is missing required columns.")
          print("Found:", reader.fieldnames)
          sys.exit(1)
  
      for row in reader:
  
          code = row["species"]
          scientific_name = row["scientific_name"]
          rate = float(row["substitution_rate_x1e9"])
  
          if code in rate_by_code:
              raise ValueError(
                  f"Duplicate tree code in rate table: {code}"
              )
  
          if rate <= 0:
              raise ValueError(
                  f"Invalid substitution rate for {code}: {rate}"
              )
  
          rate_by_code[code] = rate
          scientific_name_by_code[code] = scientific_name
  
  
  # ============================================================
  # DISCOVER CLEAN TABLES
  # ============================================================
  
  clean_files = sorted(
      p
      for p in DATA_DIR.glob("*.fa.clean.tsv")
      if not p.name.startswith("._")
  )
  
  
  if not clean_files:
      print("ERROR: no *.fa.clean.tsv files found.")
      sys.exit(1)
  
  
  # ============================================================
  # PREFLIGHT
  #
  # Nothing is written unless ALL calculable species are mapped.
  # ============================================================
  
  unmapped_assemblies = []
  missing_rates = []
  excluded = []
  calculable = []
  
  
  for clean_file in clean_files:
  
      assembly = clean_file.name.removesuffix(
          ".fa.clean.tsv"
      )
  
      if assembly not in assembly_to_tree_code:
  
          unmapped_assemblies.append(
              (assembly, clean_file.name)
          )
  
          continue
  
      tree_code = assembly_to_tree_code[assembly]
  
      if tree_code is None:
  
          excluded.append(
              (assembly, clean_file.name)
          )
  
          continue
  
      if tree_code not in rate_by_code:
  
          missing_rates.append(
              (assembly, tree_code)
          )
  
          continue
  
      calculable.append(
          (clean_file, assembly, tree_code)
      )
  
  
  print()
  print("=" * 72)
  print("PREFLIGHT")
  print("=" * 72)
  print("Real clean TSV files:", len(clean_files))
  print("Calculable:", len(calculable))
  print("Excluded because no mu:", len(excluded))
  print("Unmapped assemblies:", len(unmapped_assemblies))
  print("Missing rates:", len(missing_rates))
  print()
  
  
  if excluded:
  
      print("EXCLUDED:")
  
      for assembly, filename in excluded:
          print(
              f"  {assembly} -> Myotis tricolor "
              f"(no species-specific mu)"
          )
  
      print()
  
  
  if unmapped_assemblies:
  
      print("ERROR: these assemblies are not in the mapping:")
  
      for assembly, filename in unmapped_assemblies:
          print(" ", filename)
  
      sys.exit(1)
  
  
  if missing_rates:
  
      print("ERROR: mapped tree codes without a rate:")
  
      for assembly, tree_code in missing_rates:
          print(
              f"  {assembly} -> {tree_code}"
          )
  
      sys.exit(1)
  
  
  print("Preflight passed.")
  print("No age files have been written yet.")
  print()
  
  
  # ============================================================
  # FUNCTION: ADD INSERTION AGE
  # ============================================================
  
  def add_age_column(
      input_file,
      output_file,
      rate_x1e9
  ):
  
      rows_written = 0
  
      temp_file = Path(
          str(output_file) + ".tmp"
      )
  
      try:
  
          with input_file.open(
              "r",
              encoding="utf-8",
              newline=""
          ) as fin, temp_file.open(
              "w",
              encoding="utf-8",
              newline=""
          ) as fout:
  
              reader = csv.reader(
                  fin,
                  delimiter="\t"
              )
  
              writer = csv.writer(
                  fout,
                  delimiter="\t",
                  lineterminator="\n"
              )
  
              # ------------------------------------------------
              # Read + verify original clean header
              # ------------------------------------------------
  
              try:
                  header = next(reader)
              except StopIteration:
                  raise ValueError(
                      f"Empty file: {input_file.name}"
                  )
  
              if header != EXPECTED_HEADER:
  
                  raise ValueError(
                      f"Unexpected header in "
                      f"{input_file.name}\n"
                      f"Found:\n{header}"
                  )
  
              # ------------------------------------------------
              # Insert age immediately after perc_div
              # ------------------------------------------------
  
              div_index = header.index("perc_div")
  
              new_header = (
                  header[:div_index + 1]
                  + ["insertion_age_Ma"]
                  + header[div_index + 1:]
              )
  
              writer.writerow(new_header)
  
              # ------------------------------------------------
              # Process rows
              # ------------------------------------------------
  
              for data_row_number, row in enumerate(
                  reader,
                  start=1
              ):
  
                  if len(row) != len(EXPECTED_HEADER):
  
                      raise ValueError(
                          f"Wrong column count in "
                          f"{input_file.name}\n"
                          f"Data row: {data_row_number:,}\n"
                          f"Expected: {len(EXPECTED_HEADER)}\n"
                          f"Found: {len(row)}"
                      )
  
                  try:
                      perc_div = float(
                          row[div_index]
                      )
                  except ValueError:
  
                      raise ValueError(
                          f"Invalid perc_div in "
                          f"{input_file.name}\n"
                          f"Data row: {data_row_number:,}\n"
                          f"Value: {row[div_index]!r}"
                      )
  
                  # ============================================
                  # INSERTION AGE
                  #
                  # T = D / (100 * mu)
                  #
                  # D = perc_div
                  #
                  # mu = rate_x1e9 * 10^-9
                  #
                  # Convert years -> Ma:
                  #
                  # age_Ma =
                  #     10 * perc_div / rate_x1e9
                  # ============================================
  
                  insertion_age_Ma = (
                      10.0
                      * perc_div
                      / rate_x1e9
                  )
  
                  new_row = (
                      row[:div_index + 1]
                      + [f"{insertion_age_Ma:.6f}"]
                      + row[div_index + 1:]
                  )
  
                  if len(new_row) != 17:
                      raise ValueError(
                          f"Internal error: generated row "
                          f"does not have 17 columns in "
                          f"{input_file.name}"
                      )
  
                  writer.writerow(new_row)
  
                  rows_written += 1
  
          # Only create final file after complete success
          temp_file.replace(output_file)
  
          return rows_written
  
      except Exception:
  
          if temp_file.exists():
              temp_file.unlink()
  
          raise
  
  
  # ============================================================
  # PROCESS ALL CALCULABLE SPECIES
  # ============================================================
  
  summary = []
  failures = []
  
  start_all = time.time()
  
  
  for index, (
      clean_file,
      assembly,
      tree_code
  ) in enumerate(calculable, start=1):
  
      rate_x1e9 = rate_by_code[tree_code]
  
      scientific_name = (
          scientific_name_by_code[tree_code]
      )
  
      output_file = (
          clean_file.parent
          / (
              clean_file.name.removesuffix(
                  ".clean.tsv"
              )
              + ".age.tsv"
          )
      )
  
      print(
          f"[{index}/{len(calculable)}] "
          f"{assembly}"
      )
  
      print(
          f"    species: {scientific_name}"
      )
  
      print(
          f"    tree code: {tree_code}"
      )
  
      print(
          f"    mu x1e9: {rate_x1e9:.12g}"
      )
  
      # --------------------------------------------------------
      # Skip existing final output unless requested otherwise
      # --------------------------------------------------------
  
      if output_file.exists() and not OVERWRITE:
  
          print(
              f"    SKIPPED: {output_file.name} "
              f"already exists"
          )
  
          summary.append({
              "assembly": assembly,
              "scientific_name": scientific_name,
              "tree_code": tree_code,
              "substitution_rate_x1e9": rate_x1e9,
              "input_file": clean_file.name,
              "output_file": output_file.name,
              "status": "SKIPPED_EXISTING",
              "rows_written": "",
          })
  
          print()
          continue
  
      start_file = time.time()
  
      try:
  
          rows_written = add_age_column(
              clean_file,
              output_file,
              rate_x1e9
          )
  
          elapsed = time.time() - start_file
  
          print(
              f"    OK: {rows_written:,} rows"
          )
  
          print(
              f"    output: {output_file.name}"
          )
  
          print(
              f"    time: {elapsed / 60:.2f} min"
          )
  
          summary.append({
              "assembly": assembly,
              "scientific_name": scientific_name,
              "tree_code": tree_code,
              "substitution_rate_x1e9": rate_x1e9,
              "input_file": clean_file.name,
              "output_file": output_file.name,
              "status": "OK",
              "rows_written": rows_written,
          })
  
      except Exception as error:
  
          print("    ERROR")
          print(str(error))
  
          failures.append(
              (assembly, str(error))
          )
  
          summary.append({
              "assembly": assembly,
              "scientific_name": scientific_name,
              "tree_code": tree_code,
              "substitution_rate_x1e9": rate_x1e9,
              "input_file": clean_file.name,
              "output_file": output_file.name,
              "status": "ERROR",
              "rows_written": "",
          })
  
      print()
  
  
  # ============================================================
  # RECORD EXCLUDED SPECIES IN SUMMARY
  # ============================================================
  
  for assembly, clean_filename in excluded:
  
      summary.append({
          "assembly": assembly,
          "scientific_name": "Myotis tricolor",
          "tree_code": "",
          "substitution_rate_x1e9": "",
          "input_file": clean_filename,
          "output_file": "",
          "status": "EXCLUDED_NO_SPECIES_RATE",
          "rows_written": "",
      })
  
  
  # ============================================================
  # WRITE SUMMARY
  # ============================================================
  
  summary_file = (
      DATA_DIR
      / "repeatmasker_insertion_age_summary.tsv"
  )
  
  
  with summary_file.open(
      "w",
      encoding="utf-8",
      newline=""
  ) as fout:
  
      fieldnames = [
          "assembly",
          "scientific_name",
          "tree_code",
          "substitution_rate_x1e9",
          "input_file",
          "output_file",
          "status",
          "rows_written",
      ]
  
      writer = csv.DictWriter(
          fout,
          fieldnames=fieldnames,
          delimiter="\t",
          lineterminator="\n"
      )
  
      writer.writeheader()
      writer.writerows(summary)
  
  
  # ============================================================
  # FINAL REPORT
  # ============================================================
  
  elapsed_all = time.time() - start_all
  
  successful = sum(
      row["status"] == "OK"
      for row in summary
  )
  
  skipped_existing = sum(
      row["status"] == "SKIPPED_EXISTING"
      for row in summary
  )
  
  excluded_count = sum(
      row["status"] == "EXCLUDED_NO_SPECIES_RATE"
      for row in summary
  )
  
  
  print("=" * 72)
  print("INSERTION-AGE CALCULATION FINISHED")
  print("=" * 72)
  
  print("Clean TSV files:", len(clean_files))
  print("Successfully calculated:", successful)
  print("Already existing / skipped:", skipped_existing)
  print("Excluded (no species rate):", excluded_count)
  print("Failed:", len(failures))
  print(
      f"Total time: "
      f"{elapsed_all / 60:.2f} min"
  )
  
  print("Summary:")
  print(summary_file.resolve())
  
  
  if failures:
  
      print()
      print("FILES WITH ERRORS:")
  
      for assembly, error in failures:
          print()
          print(assembly)
          print(error)
  
      sys.exit(1)
  
  
  print()
  print("Finished without calculation errors.")
  PY

.. code-block:: bash

  python add_insertion_ages.py
