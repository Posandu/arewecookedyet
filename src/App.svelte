<script lang="ts">
	import { onDestroy, onMount } from "svelte";
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

	let akdvotes = 0;
	let sajivotes = 0;
	let loading = true;

	const load = async () => {
		loading = true;

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

				akdvotes = 0;
				sajivotes = 0;

				// for each division, update the map
				divisionVotes.forEach((division, divisionId) => {
					if (divisionId[0] !== "d") return;

					const selector = document.querySelector(`[data-id="${divisionId}"]`);
					if (selector) {
						selector.setAttribute("fill", division.winningColor);

						division.data.forEach((candidate) => {
							if (candidate.party === "ANURA KUMARA DISSANAYAKE") {
								akdvotes += candidate.votes;
							} else if (candidate.party === "SAJITH PREMADASA") {
								sajivotes += candidate.votes;
							}
						});
					}
				});

				loading = false;
			})
			.catch((error) => console.error("Error:", error));
	};

	load();

	let int: any = null;

	onMount(() => {
		int = setInterval(load, 10000);
	});

	onDestroy(() => {
		clearInterval(int);
	});
</script>

<h1>
	{akdvotes > sajivotes
		? "Not yet"
		: sajivotes - akdvotes >= 1000000
			? "We're cooked. Time to learn how to make kottu roti professionally."
			: sajivotes - akdvotes >= 500000
				? "We're cooked. Anyone know a good real estate agent in Antarctica?"
				: sajivotes - akdvotes >= 250000
					? "We're cooked. Is it too late to become a coconut picker?"
					: sajivotes - akdvotes >= 100000
						? "We're cooked. Maybe we can start a 'Disappointed Voters' support group?"
						: sajivotes - akdvotes >= 50000
							? "We're cooked. Time to practice our 'This is fine' meme face."
							: sajivotes - akdvotes >= 25000
								? "We're cooked. Anyone up for a spontaneous pilgrimage to Adam's Peak?"
								: sajivotes - akdvotes >= 10000
									? "We're cooked. Let's all agree to blame it on the Dhal curry, okay?"
									: sajivotes - akdvotes >= 5000
										? "We're cooked. Time to invest in a lifetime supply of Panadol."
										: sajivotes - akdvotes >= 1000
											? "We're cooked. Who's up for a consolation trip to Nuwara Eliya?"
											: "We're cooked. But hey, at least we still have our sense of humor, right? ...Right?"}
</h1>

<p>
	{akdvotes} votes for Anura Kumara Dissanayake
</p>

<p>
	{sajivotes} votes for Sajith Premadasa
</p>

{#if loading}
	<p>Loading...</p>
{/if}

<LkMap />
