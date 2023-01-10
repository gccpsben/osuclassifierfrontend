<template>
	<div id="mainGrid">
		<div id="navBar" class="verticallyCenter horizonalRight" style="padding-right:15px;">
			<div style="color:#FF66AA;">
				Made by 
				<a target="_blank" href="https://osu.ppy.sh/users/5988903">BrandonH</a> and 
				<a target="_blank" href="https://osu.ppy.sh/users/3206546">EMdT</a> 
				with
				<span class="material-icons" style="font-size:19px; margin-left:5px; transform: translateY(3px);">favorite</span>
			</div>
		</div>
		<div id="requirementModal" v-if="isModalOpen" class="center">
			<div style="position:fixed; top:0px; left:0px; width:100vw; height:100vh; background:#00000088; z-index:998;" @click="isModalOpen = false" ></div>
			<ubGrid id="requirementModalContent" rows="50px 1fr">
				<gridArea area="up"  style="position:relative;">
					<lrGrid columns="1fr auto">
						<div class="verticallyCenter"><h1 class="h1Min" style="font-weight:400; font-size:24px; display:block;">Edit Requirements:</h1></div>
						<div class="verticallyCenter" style="cursor:pointer;" @click="isModalOpen = false"><span class="material-icons" style="font-size:19px; margin-left:5px; color:white;">close</span></div>
					</lrGrid>
				</gridArea>
				<gridArea area="down" class="fullHeight center">
					<GridShortcut style="height:min-content; row-gap: 15px;" columns="1fr" rows="1fr 1fr 1fr 1fr 1fr 1fr" areas="'nm' 'hd' 'hr' 'dt' 'fm' 'tb'">
						<gridArea v-for="(subLabels, labelType, index) in labels" :area="labelType.split(' ')[0].toLowerCase()"  style="display:flex; gap:15px; justify-content: center;">
							<div @click="subLabel[1] = !subLabel[1]" :class="{'enabled': subLabel[1]}" v-for="subLabel in subLabels" class="labelBox"><h1>{{ subLabel[0] }}</h1></div>
						</gridArea>
					</GridShortcut>
				</gridArea>
			</ubGrid>
		</div>
		<div id="formSection">
			<GridShortcut columns="1fr" rows="min-content min-content min-content auto auto" areas="'search' 'tabs' 'tabsBorder' 'content' 'submit'" id="formContainer">
				<gridArea area="search" id="mainSearchGridContainer">
					<div id="mainSearchGrid">
						<div> <span class="material-icons">search</span> </div>
						<input @keyup.native.enter="getMaps" id="mainSearch" v-model="queryText" type="text" placeholder="Search for names, tags etc..." />
					</div>
				</gridArea>
				<gridArea area="tabs">
					<lrGrid id="formSelectorContainer" columns="1fr 1fr">
						<gridArea area="left" @click="isLabelsMode = false"><h1>Metadata</h1></gridArea>
						<gridArea area="right" @click="isLabelsMode = true"><h1>Labels</h1></gridArea>
					</lrGrid>
				</gridArea>
				<div><div id="tabBorder" :class="{'translated': isLabelsMode}"></div></div>
				
				
				<div id="formContent">

					<div v-show="!isLabelsMode">

						<GridShortcut class="statSelectorContainer" v-for="statDefinition in stats" 
						rows="1fr" columns="100px auto 1fr auto" areas="'statLabel minInput selector maxInput'">

							<gridArea area="statLabel">
								<h1>{{ statDefinition[0] }}:</h1>
							</gridArea>
							<gridArea area="minInput">
								<input @keypress="isNumber($event)" v-model.number="statDefinition[1]" class="statSelectorMinBox" type="text" :step="statDefinition[5]"/>
							</gridArea>
							<gridArea area="selector">
								<rangeSelector 
								:rangeMax="statDefinition[4]" :rangeMin="statDefinition[3]" :step="statDefinition[5]"
								v-model:max="statDefinition[2]" v-model:min="statDefinition[1]"></rangeSelector>
							</gridArea>
							<gridArea area="maxInput">
								<input @keypress="isNumber($event)" v-model.number="statDefinition[2]" class="statSelectorMaxBox" type="text" :step="statDefinition[5]"/>
							</gridArea>

						</GridShortcut>
						
					</div>

					<div v-show="isLabelsMode" style="height:100%; min-width: min(50vw, 650px);">

						<div id="condensedEditButton" v-if="enabledLabels.length != 0">
							<div @click="isModalOpen=true">
								<lrGrid columns="1fr 1fr" style="width:min-content;">
									<div class="verticallyCenter">
										<span class="material-icons">edit</span>
									</div>
									<div class="verticallyCenter">
										<h1 class="h1Min" style="font-weight:400; white-space:nowrap;">Edit Requirements...</h1>
									</div>
								</lrGrid>
							</div>
						</div> 

						<GridShortcut v-if="enabledLabels.length != 0" class="labelsSelectorContainer" v-for="label in enabledLabels" 
						rows="1fr" columns="100px auto 1fr auto" areas="'statLabel minInput selector maxInput'">

							<gridArea area="statLabel"><h1>{{ label[0] }}:</h1></gridArea>
							<gridArea area="minInput"><input @keypress="isNumber($event)" v-model.number="label[2]" class="statSelectorMinBox" type="text" :step="0.01"/></gridArea>
							<gridArea area="selector">
								<rangeSelector :rangeMax="1" :rangeMin="0" :step="0.01"
								v-model:max="label[3]" v-model:min="label[2]"></rangeSelector>
							</gridArea>
							<gridArea area="maxInput"><input @keypress="isNumber($event)" v-model.number="label[3]" class="statSelectorMaxBox" type="text" :step="0.01"/></gridArea>

						</GridShortcut>

						<div v-if="enabledLabels.length == 0" style="cursor:pointer; height:300px;" class="center fullHeight" @click="isModalOpen=true">
							<div class="fullWidth">
								<center><span class="material-icons" style="font-size:30px; margin-bottom:15px; transform: translateY(3px); color:white;">add_circle</span></center>
								<center><h1 class="h1Min" style="font-weight:400; display:block;">Edit Requirements...</h1></center>
							</div>
						</div> 

					</div>

				</div> 
				<div id="formSubmission">
					<button @click="getMaps()">Search</button>
				</div>
			</GridShortcut>
		</div>
		<div id="loadingSection" v-if="isLoading">
			<div style="position: relative; width:100%; margin-top:200px; opacity: 0.2;" class="center">
				<div id="arCircle" style=""></div>
				<div id="hitCircle" style=""></div>
				<div id="circleExpand" style=""></div>
			</div>
			<h1 class="h1Min center" style="z-index:999; position: absolute; left:50%; transform:translateX(-50%) translateY(-51%); font-weight:400;">LOADING</h1>
		</div>
		<div id="emptySection" v-if="sampleData.length == 0 && !isLoading" class="center" style="height:300px;">
			<h1 class="h1Min" style="z-index:100; font-weight:400;">No Maps Found</h1>
		</div>
		<div id="mapsSection">
			<div style="position: relative;" v-if="!isLoading">
				
				<div v-for="box in sampleData" style="position: relative;" class="mapBox" 
				:class="{'DT': box.label.includes('DT'), 'HR': box.label.includes('HR'), 'FM': box.label.includes('FM'), 'HD': box.label.includes('HD'), 'NM': box.label.includes('NM'), 'TB': box.label.includes('TB')}">
				
					<div :style="{'background-image':box.set_id == null ? `url(https://osu.ppy.sh/assets/images/default-bg.7594e945.png)` : `url(https://assets.ppy.sh/beatmaps/${box.set_id}/covers/card.jpg)`}" 
					style="background-size: cover; position:absolute;" class="fullSize"></div>
					
					<div class="mapBoxHoverAnimation"></div>
					<div @click="" class="mapBoxContent">
						<lrGrid columns="1fr 100px" rows="1fr" areas="'left right'" style="padding:15px 15px 0px 15px; color:white;">
							<gridArea area="left"><h1 class="titleText">{{ box.title }}</h1></gridArea>
							<gridArea area="right" class="horizonalRight">
								<div style="position: absolute;">
									<h1 class="h1Min" style="font-size:14px; display:inline;">{{ getBoxTopConfidence(box) }}</h1>
									<h1 class="h1Min" style="font-size:18px; display:inline;">{{ box.label }}</h1>
									<div style="margin-top:5px;">
										<h1 class="starText">{{ box.stars }}</h1>
										<span class="material-icons" style="font-size: 18px; color:var(--accent); margin-left: 5px; transform: translateY(3px); ">star</span>
									</div>
								</div>
							</gridArea>
						</lrGrid>
						<div style="position: absolute; height:40px; bottom:0; width:170px; margin:15px;">
							<GridShortcut columns="1fr 1fr 1fr 1fr 1.3fr" rows="1fr" areas="'cs ar od hp bpm'">
								<gridArea v-for="item in ['ar','cs', 'hp', 'od', 'bpm']" :area="item">
									<ubGrid rows="1fr 1fr">
										<gridArea area="up">
											<div class="horizontallyCenter">
												<h1 class="h1Min" style="font-size:12px;">{{ item.toUpperCase() }}</h1>
											</div>
										</gridArea>
										<gridArea area="down">
											<div class="horizontallyCenter">
												<h1 class="h1Min" style="font-size:16px;">{{ box[item] }}</h1>
											</div>
										</gridArea>
									</ubGrid>
								</gridArea>
							</GridShortcut>
						</div>
						<div class="tight" style="position: absolute; height:auto; bottom:0; right:0px; width:150px; margin:15px;">
							<GridShortcut class="fullSize" columns="1fr" rows="1fr 1fr 1fr 1fr">
								<div v-for="labelScorePair in mapBoxToLabelsList(box).splice(0,4)">
									<lrGrid columns="50px 1fr">
										<div>
											<div :style="{'color':labelToColorHex(labelScorePair[0])}" 
											style="font-size:8px; font-weight:400;" class="h1Min">
											{{ (labelScorePair[1] * 100).toFixed(0) }}% {{ labelScorePair[0] }}
											</div>
										</div>
										<div>
											<lrGrid class="fullSize" 
											:columns="`${getConfidenceGridPortions(box, labelScorePair[0])[0]}fr ${getConfidenceGridPortions(box, labelScorePair[0])[1]}fr`">

												<div></div>
												<div class="fullSize" style="opacity: 0.8;" :style="{'background':labelToColorHex(labelScorePair[0])}"></div>

											</lrGrid>
										</div>
									</lrGrid>
								</div>
							</GridShortcut>
						</div>
						<h1 class="diffText">{{ box.version }}</h1>
					</div>
					<gridShortcut columns="1fr" rows="1fr 1fr" areas="'bancho' 'banchodirect'" id="portals">
						
						<gridArea @click="goToPortal(portal[1], box.set_id, box.id)" v-for="portal in portals" :area="portal[1]" class="redirectOption fullWidth" :class="portal[1]">
							<lrGrid class="fullWidth" columns="1fr 40px">
								<gridArea area="left"><h2>{{ portal[0] }}</h2></gridArea>
								<gridArea area="right"><span class="material-icons">keyboard_double_arrow_right</span></gridArea>
							</lrGrid>
						</gridArea>
					
					</gridShortcut>
				</div>
			</div>
		</div>
		<div id="background"></div>

	</div>
</template>

<script>
import lrGrid from "/src/components/LRGrid.vue";
import GridShortcut from "/src/components/gridShortcut.vue";
import gridArea from "/src/components/gridArea.vue";
import rangeSelector from "/src/components/rangeSelector.vue"
import ubGrid from "/src/components/UBGrid.vue"

export default 
{
	name: 'App',
	components: 
	{
		lrGrid, GridShortcut, gridArea,rangeSelector,ubGrid
	},
	watch:
	{
		
	},
	mounted() { },
	methods:
	{
		getBoxTopConfidence(box)
		{
			return `${(box.labels[box.label.replace(" ","")] * 100).toFixed(0)}% `;
		},
		isNumber (evt) 
		{
			var keysAllowed = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '.'];
			var keyPressed = evt.key;
			
			if (!keysAllowed.includes(keyPressed)) { evt.preventDefault() }
		},
		httpPost(theUrl, callback, body)
		{
			var xmlHttp = new XMLHttpRequest();
			if (callback != undefined) xmlHttp.onreadystatechange = function ()
			{
				if (xmlHttp.readyState == 4 && xmlHttp.status == 200) callback(xmlHttp.responseText); 
			};
			xmlHttp.open("POST", theUrl, true); // false for synchronous request
			xmlHttp.setRequestHeader("Content-Type", "application/json");
			xmlHttp.send(body);
			//return xmlHttp.responseText;
		},
		getMaps()
		{
			var self = this;
			var stats = this.stats;
			var requestBody =
			{
				length: [stats[0][1], stats[0][2]],
				ar: [stats[1][1], stats[1][2]],
				od: [stats[2][1], stats[2][2]],
				hp: [stats[3][1], stats[3][2]],
				cs: [stats[4][1], stats[4][2]],
				stars: [stats[5][1], stats[5][2]],
				random: true,
				labels: self.selectedLabelsRequest,
				text: self.queryText
			};
			self.isLoading = true;
			this.httpPost("/api/maps", (data)=>
			{
				self.sampleData = JSON.parse(data);
				self.isLoading = false;
			}, JSON.stringify(requestBody));
		},
		goToPortal(protalName, setId, id)
		{
			if (protalName == "bancho")
			{
				window.open(`https://osu.ppy.sh/beatmaps/${id}`, '_blank').focus();
			}
			else if (protalName == "banchodirect")
			{
				var iframe = document.createElement('iframe');
				iframe.style.opacity = 0;
				iframe.src = `osu://b/${id}`;
				document.body.appendChild(iframe);
				iframe.contentWindow.document.open();
				iframe.contentWindow.document.close();
				iframe.outerHTML = "";
			}
		},
		mapBoxToLabelsList(mapBoxJSON)
		{
			// example output: [["NM1", 0.3], ...]
			var keysToInclude = Object.keys(mapBoxJSON.labels);
			var unorderedList = keysToInclude.map(label => { return [label, mapBoxJSON.labels[label]]; });
			var orderedList = unorderedList.sort((a,b) => { return b[1] - a[1] });
			return orderedList;
		},
		labelToColorHex(labelString)
		{
			switch (labelString.toLowerCase().replace(/[0-9]/g, ''))
			{
				case "nm":
					return "#c9c9c9";
				case "dt":
					return "#aa88ff";
				case "hr":
					return "#ed7787";
				case "tb":
					return "#ed1121";
				case "hd":
					return "#ffcc22";
				case "fm":
					return "#66ccff";
				default:
					return "red"
			}
		},
		getConfidenceGridPortions(mapJSON, label)
		{
			var labels = mapJSON.labels;
			var maxConf = Object.values(labels).sort().reverse()[0];
			var labelNormalizedScore = labels[label] / maxConf;
			console.log([1 - labelNormalizedScore, labelNormalizedScore]);
			return [1 - labelNormalizedScore, labelNormalizedScore];
		}
	},
	data()
	{
		return { 
			isLabelsMode: true,
			sampleData:[],
			queryText: "",
			stats:
			[
				// [statName, selectedMinValue, selectedMaxValue, selectableMin, selectableMax, steps]
				["Length (s)", 30, 9999, 0, 9999 , 1  ],
				["AR"        , 8 , 10  , 0, 10   , 0.1],
				["OD"        , 7 , 10  , 0, 10   , 0.1],
				["HP"        , 0 , 10  , 0, 10   , 0.1],
				["CS"        , 3 , 6   , 0, 10   , 0.1],
				["Stars"     , 3 , 10  , 0, 10   , 0.1]
			],
			labels:
			{
				// [statName, isEnabled, selectedMin, selectedMax]
				"NM":
				[
					["NM 1", false, 0.5, 1],
					["NM 2", false, 0.5, 1],
					["NM 3", false, 0.5, 1],
					["NM 4", false, 0.5, 1],
					["NM 5", false, 0.5, 1],
				],

				"HD":
				[
					["HD 1", false, 0.5, 1],
					["HD 2", false, 0.5, 1],
				],

				"HR":
				[
					["HR 1", false, 0.5, 1],
					["HR 2", false, 0.5, 1],
				],

				"FM":
				[
					["FM 1", false, 0.5, 1],
					["FM 2", false, 0.5, 1],
				],
				
				"DT":
				[
					["DT 1", false, 0.5, 1],
					["DT 2", false, 0.5, 1],
				],

				"TB":
				[
					["TB", false, 0.5, 1],
				]
			},
			portals:
			[
				[ "Beatmap Page", "bancho" ],
				[ "osu! Direct", "banchodirect" ]
			],
			isLoading: false,
			currentSongSetId: null,
			isModalOpen: false
		}
	},
	computed:
	{
		enabledLabels()
		{
			var result = [];
			var self = this;
			Object.keys(this.labels).forEach(key => { result = result.concat(self.labels[key].filter(subLabel => subLabel[1])); });
			return result;
		},
		selectedLabelsRequest()
		{
			var output = {};
			this.enabledLabels.map(label => 
			{
				var transformedLabel = [...label];
				transformedLabel[0] = transformedLabel[0].replace(" ","").toUpperCase();
				return transformedLabel;
			}).forEach(label => { output[label[0]] = [label[2], label[3]] });
			return output;
		}
	}
}
</script>

<style lang="less">
@import "/src/stylesheets/globalStyle.less";
@import url('https://fonts.googleapis.com/css2?family=Exo+2:wght@100;300;500;800&display=swap');

* { transition: 0; font-family: 'Exo 2', sans-serif; font-weight:400; }

// Basic shortcuts
.h1Min { margin:0px; padding:0px; color:white; font-size:16px; font-weight:400; }
.materialIconDisabled { opacity: 0.3; cursor:not-allowed; }
.maxCenter { display:flex; width:100%; height:100%; justify-content: center; align-items: center; }
.debug { background:rgba(255,0,0,0.2); border:0px; }

// Selectors:
a { color:#66CCFF; text-decoration: none !important; font-weight: bold; }
a:visited { color:#8866EE; }
#background { background-image: url(/src/assets/bg.png); .fullSize; z-index: -999; position: fixed; top: 0; }
#navBar { height:55px; position:static; top:0; .bg(rgba(55,20,55,0.4)); filter:drop-shadow(#000 0px 1px 4px); }
#requirementModal
{ 
	position:fixed; 
	top:0px; 
	left:0px;
	width:100vw; 
	height:100vh; 
	z-index:999; 
}
#requirementModalContent 
{ 
	width:min(500px, max(100px, 50vw)); 
	height:500px; 
	background:#FF66AA22; 
	backdrop-filter: blur(9px); 
	border:1px solid #FF66AA44; 
	border-radius: 5px; 
	padding:25px; 
	z-index:999; 
}
.diffText
{
	.h1Min; .ellipsis;
	width:70%; font-size:14px; padding:3px 0px 0px 15px; font-weight: 400;
}
.titleText
{
	.h1Min; .ellipsis; 
	width:70%; font-size:18px; font-weight: 400; width:75%;
}
.starText 
{
	.h1Min;
	font-size:14px; padding:3px 0px 0px 15px; font-weight: 400; display: inline;
}

// Selectors (organized by Sections):
#formSection
{
	margin-top:50px; padding:15px;
	min-height: 400px;
	
	.horizontallyCenter;
	
	#formContainer 
	{ 
		padding:15px; background:#FF99CC15; .border1px(#424); backdrop-filter: blur(5px); 

		#mainSearchGridContainer 
		{ 
			.horizontallyCenter; grid-area: search; border: 1px solid #282828; position: relative; height: 50px; .easeAll(0.3s); 
			&:hover { backdrop-filter: hue-rotate(30deg) brightness(2); .easeAll(0.3s); }
			#mainSearch
			{
				width: inherit;
				outline: none; grid-area: left; height:100%; padding:0px; background:transparent; color:white; border:1px solid transparent; 
				font-size:22px; padding-left: 18px;
				box-sizing: border-box;
				transition: all 0.3s ease;
			}
			#searchIcon { font-size: 24px; transform:translateY(1px); }
		}

		#formContent
		{

			.verticallyCenter;
			& > div { .fullWidth; }
		}

		#formSubmission
		{
			margin-top: 25px;
			height:50px; .center;
			& > div { .fullWidth; }
			button 
			{
				color:white; 
				.bg(#FF66AA22);
				padding:5px 10px 5px 10px;
				.border1px(#BB1177);
				cursor: pointer;
			}
			button:hover
			{
				backdrop-filter: hue-rotate(30deg) brightness(2); .easeAll(0.3s);
			}
		}
		
		.statSelectorContainer 
		{ 
			div { .verticallyCenter; }
			h1 { .h1Min; font-weight:400; }
			padding-left:15px; height:25px; gap:15px; padding-top:15px; 
			.statSelectorMinBox { width:50px; border: 1px solid #BB1177; .bg(#FF66AA12); color:white; text-align: center; padding:5px 0px 5px 0px }
			.statSelectorMaxBox { .statSelectorMinBox; margin-right:15px; }
		}

		#formSelectorContainer
		{ 
			height:35px; width:100%; margin-top:15px; padding-top:5px; 
			.tab { border-bottom:1px solid #222222; cursor: pointer; }
			.tab.selected { border-bottom:1px solid #FF66AA; }
			h1 { .verticallyCenter; .h1Min; display:inline; font-weight:400; }
			div { .horizontallyCenter; .tab; }
		}

		#tabBorder { background:#FF66AA; height:2px; width:50%; transform:translateX(0%); transition:all 0.3s ease;  }
		#tabBorder.translated { transform:translateX(100%); transition:all 0.3s ease; }

		#mainSearchGrid
		{
			& > div { grid-area: right; color:white; .maxCenter; }
			.grid(1fr 50px, 1fr, 'left right');
			.fullSize;
			.border1px(transparent);
			.easeAll(0.3s);
			&:hover { .border1px(rgb(124, 56, 124)) }
		}

		.labelsSelectorContainer 
		{ 
			&:nth-child(1) { padding-top:15px; }	
			div { .verticallyCenter; }
			h1 { .h1Min; font-weight:400; }
			padding-left:15px; gap:15px;
			.statSelectorMinBox { width:50px; border: 1px solid #BB1177; .bg(#FF66AA12); color:white; text-align: center; padding:5px 0px 5px 0px }
			.statSelectorMaxBox { .statSelectorMinBox; margin-right:15px; }
		}

		#condensedEditButton
		{
			color:white;
			&:hover { color:#ED7787; }
			& > div { .fullWidth; .center; }
			.center; .fullHeight;
			cursor:pointer; margin-bottom: 25px; margin-top: 35px; height:min-content;
			span 
			{
				font-size:20px; transform: translateY(1px); margin-right:15px; color:inherit;
			}
			h1 { color:inherit; }
		}
	}
}

.labelBox
{
	border:1px solid #FF66AA; width:minmax(25px, 5vw); height:min-content; padding:5px; border-radius: 5px;
	opacity: 0.3; cursor: pointer;
	.easeAll(0.2s);
	
	.center;

	&:hover { backdrop-filter: brightness(3); }

	h1 
	{
		.h1Min;
		font-weight:400; font-size:18px; display:block;
	}
}

.labelBox.enabled { opacity: 1; }

.circleBase { position: absolute; border:5px solid #FF66AA; width:50px; height:50px; border-radius: 999px; background-color:#FF66AA44; }
#hitCircle 
{ 
	.circleBase; 
	
	animation: loadingAnimationHit 4s linear infinite;
	@keyframes loadingAnimationHit
	{
		0% { opacity: 0; border:2px solid #FF66AA; }
		10% { opacity: 1; border:2px solid #FF66AA; }
		50% { opacity: 1; border:2px solid #FF66AA; }
		55% { opacity: 0; border:2px solid #FF66AA; }
		99% { opacity: 0; }
		100% { opacity: 0; border:0px;  }
	}
}
#arCircle 
{ 
	.circleBase; 
	background-color: transparent; border:2px solid #FF66AA;
	animation: loadingAnimation 4s linear infinite;
	@keyframes loadingAnimation 
	{
		0% { .size(300px, 300px); opacity: 0; }
		50% { .size(50px, 50px); opacity: 1; }
		55% { opacity: 0; }
		100% { opacity: 0; }
	}
}
#circleExpand
{ 
	.circleBase; 
	background-color: transparent;
	border:1px solid #FF66AA;
	animation: loadingAnimationExpand 4s linear infinite;
	@keyframes loadingAnimationExpand 
	{
		0% { .size(50px, 50px); opacity: 0; }
		49% { .size(50px, 50px); opacity: 0; }
		50% { .size(50px, 50px); opacity: 1; }
		65% { .size(150px, 150px); opacity: 0; }
		100% { .size(50px, 50px); opacity: 0; }
	}
}

#mapsSection
{
	margin-top:35px; margin-bottom:100px;
	.horizontallyCenter;

	& > div { .grid(repeat(2, 1fr), unset, unset); row-gap: 15px; column-gap: 15px; }

	.mapBox 
	{ 
		&.DT { --accent: #AA88FF; }
		&.HD { --accent: #FFCC22; }
		&.FM { --accent: #66CCFF; }
		&.NM { --accent: #c9c9c9; }
		&.HR { --accent: #ED7787; }
		&.TB { --accent: #ED1121; }

		height:150px; width:550px;

		h1 { opacity: 1; transition:all 0.2s ease; }

		.mapBoxContent
		{
			background-size: cover; position:absolute; 
			border:1px solid var(--accent);
			.fullSize;
			overflow: hidden;
			.easeAll(0.2s);
			cursor: pointer;
			h1 { color:var(--accent); }
			backdrop-filter: brightness(0.2); 
		}

		&:hover 
		{ 
			.mapBoxContent { backdrop-filter: blur(0px) brightness(0.6); .easeAll(0.2s); }
			h1
			{
				opacity: 0.2;
				transition:all 0.2s ease;
			}
		}

		& > #portals
		{
			width:0%; transform:translateY(2px); opacity: 0;
			height: calc(100% - 2px); z-index: 100; position: absolute; right:1px; transition:all 0.3s ease;
		}
		&:hover > #portals { width:35%; opacity: 1; }
		
		.redirectOption
		{
			.verticallyCenter;
			overflow:hidden; cursor:pointer;
			h2 { .h1Min; font-weight:400; padding-left:15px; white-space: nowrap; }
			.material-icons { font-size: 19px; margin-left: 5px; transform: translateY(2px); color:white; }
		}
		.redirectOption.bancho 
		{ 
			background-color:#750b4aEE; 
			&:hover { background-color:#FF66AAEE }
		}
		.redirectOption.beatconnect 
		{ 
			background-color:#804000ee; 
			&:hover { background-color:#CC6600ee }
		}
		.redirectOption.banchodirect 
		{ 
			background-color:#0b5d74EE; 
			&:hover { background-color:#1188AAEE }
		}
	}
}

@tablet: ~"only screen and (max-width: 1200px)";

@media @tablet
{
	#mapsSection > div { .grid(1fr, unset, unset); row-gap: 15px; column-gap: 15px; }
}

</style>
