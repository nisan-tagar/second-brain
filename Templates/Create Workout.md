<%*
// List of all available workout options
const workoutOptions = [
    "Push",
    "Pull",
    "Legs",
    "Calisthenics"
];

// 1. Get user selection (mobile-friendly)
const workoutType = await tp.system.suggester(
    workoutOptions,
    workoutOptions,
    false,
    "Select Workout Type"
);

if (!workoutType) {
    new Notice("Workout selection cancelled.");
    return "";
}

// 2. Define the new file's name and path
const date = tp.date.now("YYYY-MM-DD");
const newFileName = `${date} - ${workoutType}`;
const targetFolder = "Health/Fitness Logs";

// 3. Move current file to target location with new name
await tp.file.move(`${targetFolder}/${newFileName}`);

// 4. Include the template (this processes all Templater commands)
const templatePath = `Templates/Fitness/${workoutType}`;
-%>
<% tp.file.include(`[[${templatePath}]]`) %>