# LitmusGo:

[![Slack Channel](https://img.shields.io/badge/Slack-Join-purple)](https://slack.litmuschaos.io)
![GitHub Workflow](https://github.com/litmuschaos/litmus-go/actions/workflows/push.yml/badge.svg?branch=master)
[![Docker Pulls](https://img.shields.io/docker/pulls/litmuschaos/go-runner.svg)](https://hub.docker.com/r/litmuschaos/go-runner)
[![GitHub issues](https://img.shields.io/github/issues/litmuschaos/litmus-go)](https://github.com/litmuschaos/litmus-go/issues)
[![Twitter Follow](https://img.shields.io/twitter/follow/litmuschaos?style=social)](https://twitter.com/LitmusChaos)
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/5297/badge)](https://bestpractices.coreinfrastructure.org/projects/5297)
[![Go Report Card](https://goreportcard.com/badge/github.com/litmuschaos/litmus-go)](https://goreportcard.com/report/github.com/litmuschaos/litmus-go)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Flitmuschaos%2Flitmus-go.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Flitmuschaos%2Flitmus-go?ref=badge_shield)
[![YouTube Channel](https://img.shields.io/badge/YouTube-Subscribe-red)](https://www.youtube.com/channel/UCa57PMqmz_j0wnteRa9nCaw)
<br><br>

This repo consists of Litmus Chaos Experiments written in golang. The examples in this repo are good indicators of how to construct the experiments in golang: complete with steady state checks, chaosresult generation, chaos injection etc.., post chaos checks, create events and reports for observability and configure sinks for these.

**NOTE**: This repo can be viewed as an extension to the [litmuschaos/litmus](https://github.com/litmuschaos/litmus) repo. The litmus repo will also continue to be the project's community-facing meta repo housing other important project artifacts. In that sense, litmus-go is very similar to and therefore a sister repo of [litmus-python](https://github.com/litmuschaos/litmus-python) which houses examples for experiment business logic written in python.

## Litmus SDK

The Litmus SDK provides a simple way to bootstrap your experiment and helps create the aforementioned artifacts in the appropriate directory (i.e., as per the chaos-category) based on an attributes file provided as input by the chart-developer. The scaffolded files consist of placeholders which can then be filled as desired.

It generates the custom chaos experiments with some default Pre & Post Chaos Checks (AUT & Auxiliary Applications status checks). It can use the existing chaoslib (present inside /chaoslib directory), if available else It will create a new chaoslib inside the corresponding directory.

Refer [Litmus-SDK](https://github.com/litmuschaos/litmus-go/blob/master/contribute/developer-guide/README.md) for more details.

## How to get started?

Refer the [LitmusChaos Docs](https://docs.litmuschaos.io) and [Experiment Docs](https://litmuschaos.github.io/litmus/experiments/categories/contents/)

## How do I contribute?

You can contribute by raising issues, improving the documentation, contributing to the core framework and tooling, etc.

Head over to the [Contribution guide](CONTRIBUTING.md)

## License
Here is a copy of the License: [`License`](LICENSE) 

## License Status and Vulnerability Check
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Flitmuschaos%2Flitmus-go.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Flitmuschaos%2Flitmus-go?ref=badge_large)


# Updates in node-cpu-hog branch

### File chaoslib/litmus/node-cpu-hog/lib/node-cpu-hog.go
Added Metric collection for all nodes using *kubectl top nodes*

```	//go routine to print metrix for node-cpu-hog
	go func() {
		for {
			metrixNodeCPUHog()
			time.Sleep(10 * time.Second) // Adjust the interval as needed
		}
	}()

    func metrixNodeCPUHog() {

        cmd := exec.Command("kubectl", "top", "nodes", "--use-protocol-buffers")
        out, err := cmd.CombinedOutput()
        if err != nil {
            log.Errorf("[Metrics]: Error Fetching Metrices using command kubectl top nodes --use-protocol-buffers ", err)
            return
        }

        // Convert output to string
        output := string(out)

        // Split output into lines
        lines := strings.Split(output, "\n")

        // Parse lines to extract node metrics
        data := [][]string{}
        for _, line := range lines {
            if strings.TrimSpace(line) != "" && !strings.Contains(line, "NAME") {
                fields := strings.Fields(line)
                data = append(data, fields)
            }
        }

        // Print data in a table
        log.Infof("[Metrics]: CPU/Memory Utilization by nodes: ")
        table := tablewriter.NewWriter(os.Stdout)
        table.SetHeader([]string{"Node", "CPU(cores)", "CPU%", "Memory(bytes)", "Memory%"})
        for _, v := range data {
            table.Append(v)
        }
        table.Render()

    }
```
### File pkg/probe/probe.go

```

		//Print Probe Verdict in tabular form
		log.Infof("[Probe]: Probe Details: ")
		// Create a new table
		table := tablewriter.NewWriter(os.Stdout)
		// Set table headers
		table.SetHeader([]string{"Time", "ProbeName", "ProbeType", "ProbeStatus"})
		// Add log data to the table
		table.Append([]string{time.Now().Format("2006-01-02T15:04:05Z07:00"), probe.Name, probe.Type, string(probeVerdict) + " " + emoji.Sprint(":smile:")})
		// Render the table
		table.Render()

		//Print Probe Verdict in tabular form
		log.Infof("[Probe]: Probe Details: ")
		// Create a new table
		table := tablewriter.NewWriter(os.Stdout)
		// Set table headers
		table.SetHeader([]string{"Time", "ProbeName", "ProbeType", "ProbeStatus"})
		// Add log data to the table
		table.Append([]string{time.Now().Format("2006-01-02T15:04:05Z07:00"), probe.Name, probe.Type, string(probeVerdict) + " " + emoji.Sprint(":cry:")})
		// Render the table
		table.Render()
```

## Bug reported
1. While running CMD probe got below error:

```time="2024-04-24T13:11:45Z" level=info msg="[Status]: The status of Pods are as follows" Pod=node-cpu-hog-1-1713954189-probe-kr8nqt Status=Running
time="2024-04-24T13:12:06Z" level=info msg="name: Ping Probe SOT Chaos2, err: unable to get output of cmd command\n --- at /litmus-go/pkg/probe/cmdprobe.go:132                                                 (triggerSourceCmdProbe.func1) ---\nCaused by: {\"errorCode\":\"GENERIC_ERROR\",\"reason\":\"failed to create a stderr and stdout stream, command terminated with                                                 exit code 1\"}"
time="2024-04-24T13:12:06Z" level=error msg="Probe Failed, err: probes failed\n --- at /litmus-go/pkg/probe/probe.go:339 (execute) ---\nCaused by: unable to get                                                 output of cmd command\n --- at /litmus-go/pkg/probe/cmdprobe.go:132 (triggerSourceCmdProbe.func1) ---\nCaused by: {\"errorCode\":\"GENERIC_ERROR\",\"reason\":\                                                "failed to create a stderr and stdout stream, command terminated with exit code 1\"}"
```

I was running a cmd probe which was running command *ping <ip> -c 10*, and when IP was unreachable, it was terminating with exit code 1

:::info
Note: The default node metrics collection is done after every 10 sec which can be modified as per required
:::