<script lang="ts">
	import LkMap from "./LKMap.svelte";

	const SHEET_ID = "1413cnrpOKcnB_DQUZ41aiGxXVPDfCAbq2QPU5qiy11E";
	const SHEET_NAME = "Sheet1"; // Replace with your sheet name
	const URL = `https://docs.google.com/spreadsheets/d/${SHEET_ID}/gviz/tq?tqx=out:csv&sheet=${SHEET_NAME}`;

	interface ElectionData {
		"Division ID": string;
		"Candidate Name": string;
		Votes: number;
		Color: string;
		summary_text: string;
		"": string; // For the empty column at the end
	}

	function csvToJson(csv: string): ElectionData[] {
		const lines = csv.split("\n");
		const result: ElectionData[] = [];
		const headers = parseCSVLine(lines[0]);

		for (let i = 1; i < lines.length; i++) {
			if (lines[i].trim() !== "") {
				const obj: Partial<ElectionData> = {};
				const currentline = parseCSVLine(lines[i]);

				for (let j = 0; j < headers.length; j++) {
					let value: string | number = currentline[j];
					if (value !== undefined) {
						// Convert to number if possible and if the field is "Votes"
						if (
							headers[j] === "Votes" &&
							!isNaN(value.replace(/,/g, "") as any)
						) {
							value = parseFloat(value.replace(/,/g, ""));
						}
					}
					obj[headers[j] as keyof ElectionData] = value as any;
				}

				result.push(obj as ElectionData);
			}
		}

		return result;
	}

	function parseCSVLine(line: string): string[] {
		const result: string[] = [];
		let startValueIndex = 0;
		let inQuotes = false;

		for (let i = 0; i < line.length; i++) {
			if (line[i] === '"') {
				inQuotes = !inQuotes;
			} else if (line[i] === "," && !inQuotes) {
				result.push(
					line
						.substring(startValueIndex, i)
						.replace(/^"(.*)"$/, "$1")
						.trim()
				);
				startValueIndex = i + 1;
			}
		}

		// Push the last value
		result.push(
			line
				.substring(startValueIndex)
				.replace(/^"(.*)"$/, "$1")
				.trim()
		);

		return result;
	}

	let divisionVotes: Map<
		string,
		{
			winningColor: string;
			data: {
				party: string;
				votes: number;
			}[];
		}
	> = new Map();

	fetch(URL)
		.then((response) => response.text())
		.then((data) => {
			const json = csvToJson(data);

			json.forEach((row) => {
				const DIVISION_ID: string = row["Division ID"];
				const CANDIDATE_NAME = row["Candidate Name"];
				const VOTES = row.Votes;

				if (!divisionVotes.has(DIVISION_ID)) {
					divisionVotes.set(DIVISION_ID, {
						winningColor: row.Color,
						data: [],
					});
				}

				divisionVotes.get(DIVISION_ID)?.data.push({
					party: CANDIDATE_NAME,
					votes: VOTES,
				});
			});

			console.log(divisionVotes);

			// for each division, update the map
			divisionVotes.forEach((division, divisionId) => {
				if (divisionId[0] !== "d") return;

				const selector = document.querySelector(`[data-id="${divisionId}"]`);
				if (selector) {
					selector.setAttribute("fill", division.winningColor);
				}
			});
		})
		.catch((error) => console.error("Error:", error));
</script>

<LkMap />
