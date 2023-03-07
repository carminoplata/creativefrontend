<script>
	import { Form, Step } from "svelte-multistep-form";
	import Button, { Label } from '@smui/button';
	import HelperText from '@smui/textfield/helper-text';
	import Textfield from '@smui/textfield';
	import LinearProgress from '@smui/linear-progress';
	import Select, { Option } from '@smui/select';
	import Checkbox from '@smui/checkbox';
	import FormField from '@smui/form-field';
	
	let multiStepOptions = {
  		formTitle: "The path towards Creative",
  		formSubtitle: "Fill up all steps to find the best Creative to develop your business idea",
  		stepsDescription: [
  		  { title: "STEP 1", subtitle: "Describe briefly what you want to develop. What's your digital idea?" },
  		  { title: "STEP 2", subtitle: "Skills and Roles" },
		  { title: "STEP 3", subtitle: "Personal Data" }
  		],
	};

	let FormComponentRef;
	let companyName = '';
	let companyWebsite = '';
	let fullName = '';
	let email = '';
	let role = '';
	let rolesObj = {};
	let problem = '';
	let sectors=['Aerospace', 'Fashion', 'Medical', 'Automotive', 'Consultancy', 
         		 'Digital Transformation', 'CyberSecurity', 'Oil and gas',
         		 'Energy', 'Pharmaceutical', 'Defense', 'Software'];
	let industry='';
	let roles;
	let loading = false;
	let rolesText = '';
	let skillsList = [];
	let skillsText = '';
	let tasksText = '';
	let resetSteps = false;

	
	let checked = false;

	function makeID(length) {
		let result = '';
		const characters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
		const charactersLength = characters.length;
		let counter = 0;
		while (counter < length) {
			result += characters.charAt(Math.floor(Math.random() * charactersLength));
			counter += 1;
		}
		return result;
	}

	function onSubmitClick() {
		if (companyName.length > 0 && companyWebsite.length > 0 && fullName.length > 0 && role.length > 0 && problem.length > 0) {
			let data = {
				"slug": makeID(30),
				"company": companyName,
				"email": email,
				"fullname":  fullName,
				"industry": industry,
				"problem": problem,
				"resources": rolesObj['resources'],
				"roles": rolesObj['roles'],
				"skills": skillsList,
				"tasks": rolesObj['tasks']
			};

			fetch('http://127.0.0.1:8000/postJobOffer', {
				headers: {
					'Content-Type': 'application/json',
					'test': 'test'
				},
				method: 'POST',
				body: JSON.stringify(data),
			}).then((response) => response.json()).then((result) => {
				console.log('Success:', result);
			}).catch((error) => {
				console.error('Error:', error);
			});
		}
	}

	async function findRolesForIdea(){
		if(problem.length > 10 && industry!='')
		{
			console.log("Problem" + problem + " Industry" + industry)
			let formData = new FormData();
			formData.append('problem', problem);
			formData.append('industry', industry);
			const res = await fetch('https://carminoplata.eu.pythonanywhere.com/findRolesAndSkills', {
				method: 'POST',
				body: formData
			});
			const text = await res.text();
			console.log("Received text"+text);
			if (res.ok) {
				return text;
			} else {
				throw new Error(text);
			}
		}		
	}

	async function runRolesDetection() {
		console.log("Clicked on Button")

		loading = true;
		
		
		roles = await findRolesForIdea();
		rolesObj = JSON.parse(roles);
		

		/*
		roles = {
			"resources":[
				"Web developer",
				"Machine learning engineer",
				"Database administrator",
				"User interface designer",
				"Quality assurance specialist"
			],
			"roles":" Backend Engineer, Designer, Android Developer, iOS Developer",
			"skills":[
				{
					"Web development":[
						"HTML",
						"CSS",
						"JavaScript",
						"PHP"
					]
				},
				{
					"Machine learning":[
						"Python",
						"TensorFlow"
					]
				},
				{
					"Database management":[
						"MySQL",
						"MongoDB"
					]
				},
				{
					"User interface design":[
						"UX/UI design",
						"wireframing"
					]
				},
				{
					"Quality assurance":[
						"testing",
						"debugging"
					]
				}
			],
			"tasks":[
				"Design and develop the web application",
				"Implement machine learning algorithms for automatic translation",
				"Set up and maintain the database",
				"Design the user interface",
				"Test and debug the application"
			]
		};
		*/
		/*
		roles = {
			"resources": [
				"Web developer",
				"Machine learning engineer",
				"Database engineer",
				"UI/UX designer",
				"Quality assurance engineer"
			],
			"roles": " Fullstack Developer, Designer, Android Developer, iOS Developer",
			"skills": [
				"Web development",
				"Machine learning",
				"Database design and development",
				"UI/UX design",
				"Quality assurance"
			],
			"tasks": [
				"Design and develop the web application",
				"Implement machine learning algorithms for automatic translation of recipes",
				"Design and develop the database to store the recipes",
				"Design the user interface and user experience",
				"Test the application for quality assurance"
			]
		};
		rolesObj = roles;
		*/
		
		rolesText = rolesObj['roles'];

		skillsText = '';
		for (let i of rolesObj['skills']) {
			if (typeof i === "object") {
				for (let key in i) {

				}
			} else {
				console.log(i);
				skillsList.push(i);
				skillsText += i + "\n";
			}
		}
		skillsText = skillsText.slice(0, -1);

		tasksText = '';
		for (let i of rolesObj['tasks']) {
			tasksText += i + "\n"
		}
		tasksText = tasksText.slice(0, -1);

		console.log(rolesText);
		console.log(skillsText);
		console.log(tasksText);

		loading = false;

		FormComponentRef.nextStep();
	}
</script>

<svelte:head>
	<!-- Fonts -->
	<link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
	<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,600,700">
	
	<!-- SMUI -->
	<link rel="stylesheet" href="https://unpkg.com/svelte-material-ui/bare.css" />

	<title>Hire a Creative</title>
	<meta name="description" content="Hire Your Creative and make your dream" />
</svelte:head>

<div class="text-column">
	<div id="hireForm">
		<Form {multiStepOptions} bind:this={FormComponentRef} bind:resetSteps>
			<Step> 
				<Textfield textarea variant="outlined" bind:value={problem} 
					label="Describe briefly your business Idea">
					<HelperText slot="helper">Tell us your idea...</HelperText>
				</Textfield>
				<Select class="shaped-outlined" variant="outlined" bind:value={industry}
					label="Choose your company's industry" required>
					{#each sectors as sector}
						<Option value={sector}>{sector}</Option>
					{/each}
				</Select>
				{#if loading}
					<LinearProgress indeterminate style="margin-top: 25px;" />
				{:else}
					<Button on:click={runRolesDetection} variant="raised">
						<Label>Find Roles and Skills</Label>
					</Button>
				{/if}
			</Step>
			<Step> 
				<Textfield textarea variant="outlined" bind:value={rolesText} 
					label="Roles you need to achieve your target">
				</Textfield>
				<Textfield textarea variant="outlined" bind:value={skillsText} input$rows={(skillsText.match(/\n/g) || []).length + 1}
					label="Describe briefly your business Idea">
				</Textfield>
				<Textfield textarea variant="outlined" bind:value={tasksText} input$rows={(tasksText.match(/\n/g) || []).length + 1}
					label="Describe briefly your business Idea">
				</Textfield>
				<Button on:click={() => FormComponentRef.nextStep()} variant="raised">
					<Label>Continue</Label>
				</Button>
			</Step>
			<Step>
				<Textfield textarea variant="outlined" bind:value={companyName} label="Company's name">
				</Textfield>
				<Textfield textarea variant="outlined" bind:value={companyWebsite} label="Company's website">
				</Textfield>
				<Textfield textarea variant="outlined" bind:value={fullName} label="Full name">
				</Textfield>
				<Textfield textarea variant="outlined" bind:value={email} label="Email">
				</Textfield>
				<Textfield textarea variant="outlined" bind:value={role} label="Role in the company">
				</Textfield>
				<FormField>
					<Checkbox bind:checked touch />
					<span slot="label">I accept our<a href="https://tinyurl.com/privacypolicyencreativelink"> T&C</a></span>
				</FormField>
				{#if checked}
					<Button on:click={() => onSubmitClick()} variant="raised">
						<Label>Submit</Label>
					</Button>
				{:else}
					<Button on:click={() => onSubmitClick()} variant="raised" disabled>
						<Label>Submit</Label>
					</Button>
				{/if}
			</Step>
		</Form>
	</div>
	<!--
	{#await roles}
		<p>...waiting</p>
	{:then text}
		<p>The number is {text}</p>
	{:catch error}
		<p style="color: red">{error.message}</p>
	{/await}
	-->
</div>

<style>
	@import url("https://stackpath.bootstrapcdn.com/bootstrap/4.5.0/css/bootstrap.min.css");
	.text-column :global(#hireForm h1){
		text-align: center;
		color: #007FAA;
		font-family: "Quicksand", "Open Sans", Roboto, sans-serif;
	}
	
	.text-column :global(#hireForm h5){
		text-align: center;
		color: #007FAA;
	}

	.text-column :global(#hireForm span){
		color: #007FAA;
	}
	
	.text-column :global(#hireForm .separator-check-current){
		width: 53px;
		height: 49px;
		background: #004055;
	}

	.text-column :global(#hireForm .separator-check-pending){
		width: 53px;
		height: 49px;
	}

	.text-column :global(#hireForm .mdc-text-field--textarea){
		width: 100%;
	}

	.text-column :global(#hireForm .mdc-text-field--textarea span){
		color: #212529;
	}

	.text-column :global(#hireForm .mdc-select--outlined){
		width: 100%;
	}

	.text-column :global(#hireForm .mdc-button--raised){
		margin-top: 2em;
		background-color: #00BFFF;
		font-size: 20px;
		font-family: 'Poppins', 'Quicksand', sans-serif;
		font-weight: 600;
	}

	.text-column :global(#hireForm .mdc-button--raised span){
		color: white;
	}
</style>