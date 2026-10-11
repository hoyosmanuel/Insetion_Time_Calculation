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
