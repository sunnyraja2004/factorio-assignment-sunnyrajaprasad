# Run samples
python part2_assignment/run_samples.py "python part2_assignment/factory/main.py" "python part2_assignment/belts/main.py"

# Run tests
FACTORY_CMD="python part2_assignment/factory/main.py" BELTS_CMD="python part2_assignment/belts/main.py" pytest -q
