```mermaid
stateDiagram-v2
	[*] --> Idle
	Idle --> Dash: Initiate Dash
	Dash --> Running: End of Dash
	Dash --> Idle: Stop Moving
	Running --> Idle: Stop Moving
	
	Idle --> Airborne: Jump
	Dash --> Airborne: Jump
	Running --> Airborne: Jump
	
	Idle --> Attack: Attack
	Dash --> Attack: Attack
	Running --> Attack: Attack
	Airborne --> Attack: Attack
	
	Idle --> Airdash: Airdash
	Running --> Airdash: Airdash
	Dash --> Airdash: Airdash
	Airborne --> Airdash: Airdash
	Attack --> Airdash: Airdash
```