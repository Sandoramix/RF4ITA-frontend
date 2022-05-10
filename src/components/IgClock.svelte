<script>
	const timeratio = 2.5;
	const igMinuteTimer = 2.5;
	var hour = `00`,
		time = `00`;

	function getIgTime() {
		let server_time = new Date();

		let serverMinutes = server_time.getUTCMinutes(),
			serverSeconds = server_time.getUTCSeconds(),
			totalSeconds = serverMinutes * 60 + serverSeconds;

		let ratio = totalSeconds / timeratio;

		let _igHour = Math.trunc(ratio / 60),
			_igMinute = Math.trunc(ratio - _igHour * 60);

		hour = formatTime(_igHour);
		time = formatTime(_igMinute);

		setTimeout(() => {
			getIgTime();
		}, igMinuteTimer * 1000);
	}

	const formatTime = (x) => x.toString().padStart(2, `0`);

	getIgTime();
</script>

<div class="clock">
	<div id="hour">{hour}</div>
	<div>:</div>
	<div id="minute">{time}</div>
</div>

<style>
	.clock {
		display: flex;
		color: var(--orange-light);
	}
</style>
