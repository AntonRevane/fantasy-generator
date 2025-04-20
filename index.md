<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fantasy World Generator</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/@tailwindcss/browser@latest"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
    </style>
</head>
<body class="bg-gray-100 p-6">
    <div class="container mx-auto p-8 bg-white rounded-lg shadow-md">
        <h1 class="text-3xl font-semibold text-blue-600 text-center mb-8">Welcome to the Fantasy World Generator!</h1>
        <p class="text-lg text-gray-700 text-center mb-6">
            This tool creates unique, procedurally generated worlds, where everything from the terrain to the factions
            is influenced by five core elements: Water, Fire, Grass, Holy, and Dark.
        </p>

        <div class="mb-6">
            <label for="theme-select" class="block text-gray-700 text-sm font-bold mb-2">Please select a theme:</label>
            <select id="theme-select"
                    class="shadow appearance-none border rounded w-full py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
                <option value="Dragonlords">Dragonlords</option>
                <option value="Clockwork">Clockwork</option>
                <option value="GothicHorror">GothicHorror</option>
            </select>
        </div>

        <div class="mb-6">
            <label for="race-select" class="block text-gray-700 text-sm font-bold mb-2">Select a Race and Element:</label>
            <div class="flex space-x-4">
                <select id="race-select"
                        class="shadow appearance-none border rounded w-1/2 py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
                    <option value="Elves">Elves</option>
                    <option value="Tieflings">Tieflings</option>
                    <option value="Goblins">Goblins</option>
                    <option value="Tritons">Tritons</option>
                    <option value="Aasimar">Aasimar</option>
                </select>
                <select id="element-select"
                        class="shadow appearance-none border rounded w-1/2 py-3 px-4 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
                    <option value="Holy">Holy</option>
                    <option value="Dark">Dark</option>
                    <option value="Fire">Fire</option>
                    <option value="Water">Water</option>
                    <option value="Grass">Grass</option>
                </select>
            </div>
        </div>

        <div class="mb-8">
            <h2 class="text-2xl font-semibold text-gray-800 mb-4">Factions & Races</h2>
            <p class="text-gray-700 text-base">
                Factions are influenced by these elements and the chosen theme. Each faction controls 1-3 settlements or camps.
            </p>
            <p class="text-gray-700 text-base mb-4">
                Races can be influenced by other elements to create new faction variations, resulting in subraces.
            </p>
        </div>

        <button id="generate-button"
                class="bg-gradient-to-r from-green-400 to-blue-500 hover:from-green-500 hover:to-blue-600 text-white font-bold py-2 px-4 rounded-full shadow-md focus:outline-none focus:shadow-outline transition duration-300 ease-in-out">
            Generate World
        </button>

        <div id="output" class="mt-8 p-6 bg-gray-50 rounded-lg shadow-inner">
            <h3 class="text-xl font-semibold text-gray-800 mb-4">Generated World:</h3>
            <p class="text-gray-700" id="world-description">Select a theme and click "Generate World" to see the
                result!</p>
        </div>
    </div>

    <script>
        const themeSelect = document.getElementById('theme-select');
        const generateButton = document.getElementById('generate-button');
        const output = document.getElementById('output');
        const worldDescription = document.getElementById('world-description');
        const raceSelect = document.getElementById('race-select');
        const elementSelect = document.getElementById('element-select');

        generateButton.addEventListener('click', () => {
            const selectedTheme = themeSelect.value;
            const selectedRace = raceSelect.value;
            const selectedElement = elementSelect.value;
            const generatedWorld = generateWorld(selectedTheme, selectedRace, selectedElement);
            worldDescription.innerHTML = generatedWorld;
            output.classList.remove('hidden');
        });

        function generateWorld(theme, selectedRace, selectedElement) {
            const factions = generateFactions(theme, selectedRace, selectedElement);
            const terrain = generateTerrain(theme);

            let worldDetails = `<h4 class="text-lg font-semibold text-green-600 mb-2">Theme: ${theme}</h4>`;
            worldDetails += `<h5 class="text-md font-semibold text-gray-800 mt-4 mb-2">Terrain:</h5> <p class="text-gray-700">${terrain}</p>`;
            worldDetails += `<h5 class="text-md font-semibold text-gray-800 mt-4 mb-2">Factions:</h5> <ul class="list-disc list-inside">`;
            factions.forEach(faction => {
                const subraceName = generateSubraceName(faction.race, faction.elements);
                const displayElements = faction.raceElement !== faction.elements
                    ? `${faction.raceElement}/${faction.elements}`
                    : faction.raceElement;
                worldDetails += `<li class="text-gray-700">${faction.name} - ${subraceName} (${displayElements}) - ${faction.description}</li>`;
            });
            worldDetails += `</ul>`;

            return worldDetails;
        }

        function generateTerrain(theme) {
            const baseTerrain = "Varied terrain including forests, mountains, rivers, and plains.";
            const terrainVariations = {
                Dragonlords: " with volcanic regions and dragon lairs.",
                Clockwork: " with mechanized areas and artificial structures.",
                GothicHorror: " with dark forests, haunted ruins, and eerie swamps.",
            };

            const themeTerrain = terrainVariations[theme] || "";
            const randomness = Math.random() < 0.8;
            return randomness
                ? baseTerrain + themeTerrain
                : baseTerrain;
        }

        function generateFactions(theme, selectedRace, selectedElement) {
            const baseRaces = generateRaces();
            const elementalModifiers = ["Holy", "Dark", "Fire", "Water", "Grass"];
            const factions = [];
            const factionCounts = {};
            const generatedCombinations = new Set();
            const numFactions = Math.floor(Math.random() * 3) + 5;

            baseRaces.forEach(baseRace => {
                factionCounts[baseRace.name] = 0;
            });

            let factionIndex = 0;
            let forcedFaction = false;
            while (factions.length < numFactions && factionIndex < 20) {
                let raceIndex;
                let baseRace;
                let randomModifier;
                let dragonRelationship = "";
                let clockworkAffiliation = "";
                let gothicHorrorAlignment = "";

                if (!forcedFaction && factionIndex === 0) {
                    baseRace = baseRaces.find(race => race.name === selectedRace);
                    randomModifier = selectedElement;
                    forcedFaction = true;
                } else {
                    raceIndex = Math.floor(Math.random() * baseRaces.length);
                    baseRace = baseRaces[raceIndex];
                    randomModifier = elementalModifiers[Math.floor(Math.random() * elementalModifiers.length)];
                }

                const combination = `${baseRace.name}-${randomModifier}`;
                if (generatedCombinations.has(combination)) {
                    factionIndex++;
                    continue;
                }

                if (theme === "Dragonlords") {
                    const relationshipRoll = Math.random();
                    if (relationshipRoll < 0.3) {
                        dragonRelationship = " Servants of a Dragon Lord. ";
                    } else if (relationshipRoll < 0.6) {
                        dragonRelationship = " Mortal enemies of Dragons. ";
                    } else {
                        dragonRelationship = " Riders of Dragons. ";
                    }
                }

                if (theme === "Clockwork") {
                    const clockworkRoll = Math.random();
                    if (clockworkRoll < 0.3) {
                        clockworkAffiliation = " View ancient artifacts as cursed and heretical. ";
                    } else if (clockworkRoll < 0.6) {
                        clockworkAffiliation = " Revere ancient artifacts, integrating them into life and limb. ";
                    } else {
                        clockworkAffiliation = " Use ancient tech pragmatically. ";
                    }
                }

                if (theme === "GothicHorror") {
                    const raceElement = baseRace.element;

                    if (randomModifier === "Neither") {
                        gothicHorrorAlignment = " Made an uneasy bargain with the Dark Forces. ";
                    }
                    else if (randomModifier === "Holy" && raceElement !== "Dark") {
                        gothicHorrorAlignment = " Struggles against the Dark Forces. ";
                    } else if (randomModifier === "Dark" || raceElement === "Dark") {
                        gothicHorrorAlignment = " Are loyal servants of the Dark Forces. ";
                    }
                     else if (randomModifier === "Holy/Dark" || randomModifier === "Dark/Holy") {
                        if (randomModifier.includes("Holy")) {
                            gothicHorrorAlignment = " Struggles against the Dark Forces. ";
                        }
                        else {
                            gothicHorrorAlignment = " Are loyal servants of the Dark Forces. ";
                        }
                    }
                    else {
                        gothicHorrorAlignment = " Made an uneasy bargain with the Dark Forces. ";
                    }
                }

                if (factionCounts[baseRace.name] < 3) {
                    const primaryElementFaction = {
                        name: generateFactionName(baseRace.name, randomModifier, theme, baseRace.name, gothicHorrorAlignment, dragonRelationship, clockworkAffiliation),
                        elements: randomModifier,
                        description: generateFactionDescription(baseRace.name, randomModifier, theme, baseRace.name) + dragonRelationship + clockworkAffiliation + gothicHorrorAlignment,
                        race: baseRace.name,
                        raceElement: baseRace.element,
                    };
                    factions.push(primaryElementFaction);
                    factionCounts[baseRace.name]++;
                    generatedCombinations.add(combination);
                }
                factionIndex++;
            }
            return factions;
        }

        function generateSubraceName(baseRaceName, modifier) {
            const subraceNames = {
                Elves: {
                    Holy: "High Elves",
                    Dark: "Dark Elves",
                    Fire: "Mountain Elves",
                    Water: "Sea Elves",
                    Grass: "Wood Elves",
                },
                Tieflings: {
                    Holy: "Paragon Tieflings",
                    Dark: "Abyssal Tieflings",
                    Fire: "Infernal Tieflings",
                    Water: "Deepsea Tieflings",
                    Grass: "Wild Tieflings",
                },
                Goblins: {
                    Holy: "Blessed Goblins",
                    Dark: "Night Goblins",
                    Fire: "Fire Goblins",
                    Water: "Bog Goblins",
                    Grass: "Jungle Goblins",
                },
                Tritons: {
                    Holy: "Coral Tritons",
                    Dark: "Abyssal Tritons",
                    Fire: "Volcanic Tritons",
                    Water: "Deep Tritons",
                    Grass: "Mangrove Tritons",
                },
                Aasimar: {
                    Holy: "Seraphim Aasimar",
                    Dark: "Fallen Aasimar",
                    Fire: "Fire Aasimar",
                    Water: "Aquatic Aasimar",
                    Grass: "Forest Aasimar"
                }
            };
            return subraceNames[baseRaceName][modifier];
        }

        function generateFactionName(baseRaceName, modifier, theme, raceName, gothicHorrorAlignment, dragonRelationship, clockworkAffiliation) {
            const namePrefixes = {
                Holy: ["Celestial", "Sacred", "Divine", "Radiant", "Blessed"],
                Dark: ["Shadow", "Night", "Abyssal", "Nether", "Sinister"],
                Fire: ["Blazing", "Infernal", "Ember", "Scorching", "Fiery"],
                Water: ["Aquatic", "Tidal", "Riptide", "Azure", "Abyssal"],
                Grass: ["Verdant", "Sylvan", "Emerald", "Thorned", "Wild"],
            };

            const baseNameAdjectives = {
                Elves: ["Ancient", "Mystical", "Graceful", "Enchanted", "Wise"],
                Tieflings: ["Cunning", "Shadowy", "Passionate", "Fiendish", "Intrepid"],
                Goblins: ["Wily", "Resourceful", "Mischievous", "Savage", "Tenacious"],
                Tritons: ["Sovereign", "Majestic", "Abyssal", "Tempestuous", "Resolute"],
                Aasimar: ["Righteous", "Luminous", "Seraphic", "Exalted", "Virtuous"],
            };

            const nameSuffixes = {
                Holy: ["Guardians", "Crusaders", "Order", "Host", "Sentinels"],
                Dark: ["Legion", "Cabal", "Cult", "Brood", "Syndicate"],
                Fire: ["Legion", "Horde", "Conclave", "Host", "Fury"],
                Water: ["Tide", "Dominion", "Conclave", "Host", "Navy"],
                Grass: ["Conclave", "Circle", "Host", "Wardens", "Enclave"],
                Dragonlords: ["Dragonriders", "Dragonlords", "Wyrmspeakers", "Flamebringers", "Skysovereigns"],
                Clockwork: ["Cogsmiths", "Mechanists", "Gearforged", "Automatons", "Clockworks"],
                GothicHorror: ["Nightwalkers", "Gloomseekers", "Duskdwellers", "Shadowkin", "Vampires"]
            };

            let themePrefix = "";
            let themeSuffix = "";

            if (theme === "Dragonlords") {
                if (dragonRelationship.includes("Dragon Lord")) {
                    themePrefix = "Dragonlord ";
                    themeSuffix = " " + nameSuffixes.Dragonlords[Math.floor(Math.random() * nameSuffixes.Dragonlords.length)];
                } else if (dragonRelationship.includes("enemies")) {
                    themePrefix = "Dragonbane ";
                } else {
                    themeSuffix = " " + nameSuffixes.Dragonlords[Math.floor(Math.random() * nameSuffixes.Dragonlords.length)];
                }
            }

            if (theme === "Clockwork") {
                if (clockworkAffiliation.includes("cursed")) {
                    themePrefix = "Techbane ";
                } else if (clockworkAffiliation.includes("Revere")) {
                    themePrefix = "Techpriest ";
                    themeSuffix = " " + nameSuffixes.Clockwork[Math.floor(Math.random() * nameSuffixes.Clockwork.length)];
                }
                 else {
                    themeSuffix = " " + nameSuffixes.Clockwork[Math.floor(Math.random() * nameSuffixes.Clockwork.length)];
                }
            }

            if (theme === "GothicHorror") {
                if (gothicHorrorAlignment.includes("Struggle")) {
                    themePrefix = "Lightbringer ";
                    themeSuffix = " " + nameSuffixes.GothicHorror[Math.floor(Math.random() * nameSuffixes.GothicHorror.length)];
                } else if (gothicHorrorAlignment.includes("loyal")) {
                    themePrefix = "Darkblood ";
                    themeSuffix = " " + nameSuffixes.GothicHorror[Math.floor(Math.random() * nameSuffixes.GothicHorror.length)];
                } else {
                    themeSuffix = " " + nameSuffixes.GothicHorror[Math.floor(Math.random() * nameSuffixes.GothicHorror.length)];
                }
            }


            const prefix = namePrefixes[modifier] ? namePrefixes[modifier][Math.floor(Math.random() * namePrefixes[modifier].length)] : "";
            const baseAdjective = baseNameAdjectives[raceName] ? baseNameAdjectives[raceName][Math.floor(Math.random() * baseNameAdjectives[raceName].length)] : "";
            const suffix = nameSuffixes[modifier] ? nameSuffixes[modifier][Math.floor(Math.random() * nameSuffixes[modifier].length)] : "";

            let finalName = "";
            if (Math.random() < 0.5) {
                finalName = `${themePrefix}${prefix} ${baseAdjective}${themeSuffix}`;
            }
            else {
                 finalName = `${themePrefix}${prefix} ${suffix}${themeSuffix}`;
            }

            // Trim extra spaces
            return finalName.trim();
        }

        function generateFactionDescription(baseRaceName, modifier, theme, raceName) {
            const baseDescriptions = {
                Elves: "An ancient and mystical race.",
                Tieflings: "Descendants of fiends, often cunning and resourceful.",
                Goblins: "Cunning and resourceful, often dwelling in tribes.",
                Tritons: "Aquatic humanoids, guardians of the seas.",
                Aasimar: "Descendants of celestials, often righteous and charismatic.",
            };

            const modifierDescriptions = {
                Holy: "devoted to divine powers and righteousness.",
                Dark: "steeped in shadow and dark magic.",
                Fire: "imbued with fiery passion and power.",
                Water: "connected to the power of the seas and tides.",
                Grass: "deeply connected to the natural world.",
            };

            const themeAdjustments = {
                GothicHorror: (description) => description + "",
            };

            let description = `${baseDescriptions[raceName]} `;
            if (baseRaceName !== modifier) {
                description += `They are ${modifierDescriptions[modifier]}`;
            }
            description = themeAdjustments[theme] ? themeAdjustments[theme](description) : description;
            return description;
        }

        function generateRaces() {
            return [
                { name: "Elves", element: "Grass", description: "Graceful and long-lived." },
                { name: "Tieflings", element: "Dark", description: "Descendants of fiends." },
                { name: "Goblins", element: "Fire", description: "Cunning and resourceful." },
                { name: "Tritons", element: "Water", description: "Aquatic humanoids." },
                { name: "Aasimar", element: "Holy", description: "Descendants of celestials." },
            ];
        }
    </script>
</body>
</html>
