---
title: "Train of thought writing test cases for Linux kernel's AMDGPU with KUnit"
date: 2022-08-08T00:00:00-03:00
draft: false
description: All aboard 
---
There are many ways to approach writing test cases, one of them being writing
test cases that cover discovered bugs. In this post, I'll walk through my train
of thoughts into writing one test case for a bug in the Linux kernel's AMDGPU
Display Mode Library.

# Background story

The file of interest here is located inside the Linux kernel repository at
`drivers/gpu/drm/amd/display/dc/dml/dcn20/dcn20_fpu.c`. To try and understand
the approach into building the test case, take a look at the [patch series that
fixes the
issue](https://patchwork.freedesktop.org/patch/488777/?series=104903&rev=1).

Before commit `8861c27a`, the function `dcn21_update_bw_bounding_box` worked
fine, but its 2192-byte frame size triggered a warning for exceeding the stack
size that could be used. An attempt to fix this problem is made by commit
`8861c27a`, which removes a very large variable from the function. Although this
solution does fix the stack size problem, the new version of
`dcn21_update_bw_bounding_box` is not functionally equivalent to the old one.
Then comes commit `9ad5d02c` tackling the two previous issues: the stack size no
longer exceeds the specified limit, and `dcn21_update_bw_bounding_box` now
presents the same behavior as it did before `8861c27a`.

# Setup and playing a bit

To get a grasp of what was accomplished, it could be desirable to clone the
repository and play around a bit.

```bash
git clone --branch for-amd https://gitlab.freedesktop.org/isinyaaa/linux.git
```

At the moment of writing this post, the commit that introduces the test case is
`3013c453`. For reproducibility, run:

```bash
git checkout 3013c453
```

Run the command below to run all KUnit's AMDGPU tests, they are all expected to
run successfully:

```bash
./tools/testing/kunit/kunit.py run --arch=x86_64 --kunitconfig=drivers/gpu/drm/amd/display/tests
```

Roll back `drivers/gpu/drm/amd/display/dc/dml/dcn20/dcn20_fpu.c` to the commit
version that introduced the issue:

```bash
git checkout 8861c27a -- drivers/gpu/drm/amd/display/dc/dml/dcn20/dcn20_fpu.c
```

When running the KUnit tests again, it is natural and expected that the test
will fail:

```bash
./tools/testing/kunit/kunit.py run --arch=x86_64 --kunitconfig=drivers/gpu/drm/amd/display/tests
```

Roll back `drivers/gpu/drm/amd/display/dc/dml/dcn20/dcn20_fpu.c` to a point
before the problematic commit was introduced:

```bash
git checkout 8861c27a^ -- drivers/gpu/drm/amd/display/dc/dml/dcn20/dcn20_fpu.c
```

Finally run the KUnit tests again, and notice that they all pass:

```bash
./tools/testing/kunit/kunit.py run --arch=x86_64 --kunitconfig=drivers/gpu/drm/amd/display/tests
```

# Writing a test case for dcn21_update_bw_bounding_box

The task at hand is to write a test case that fails when the file of the
function to be tested is at version `8861c27a`. Run the command below to have
only `dcn20_fpu.c` at that version.

```bash
git checkout 8861c27a -- drivers/gpu/drm/amd/display/dc/dml/dcn20/dcn20_fpu.c
```

If everything worked correctly, the function below is the same as the one in
`dcn20_fpu.c`. Now, the next step is getting a gist of the function.

```c
void dcn21_update_bw_bounding_box(struct dc *dc, struct clk_bw_params *bw_params)
{
	struct dcn21_resource_pool *pool = TO_DCN21_RES_POOL(dc->res_pool);
	struct clk_limit_table *clk_table = &bw_params->clk_table;
	unsigned int i, closest_clk_lvl = 0, k = 0;
	int j;

	dc_assert_fp_enabled();

	dcn2_1_ip.max_num_otg = pool->base.res_cap->num_timing_generator;
	dcn2_1_ip.max_num_dpp = pool->base.pipe_count;
	dcn2_1_soc.num_chans = bw_params->num_channels;

	ASSERT(clk_table->num_entries);
	/* Copy dcn2_1_soc.clock_limits to clock_limits to avoid copying over null states later */
	for (i = 0; i < dcn2_1_soc.num_states + 1; i++) {
		dcn2_1_soc.clock_limits[i] = dcn2_1_soc.clock_limits[i];
	}

	for (i = 0; i < clk_table->num_entries; i++) {
		/* loop backwards*/
		for (closest_clk_lvl = 0, j = dcn2_1_soc.num_states - 1; j >= 0; j--) {
			if ((unsigned int) dcn2_1_soc.clock_limits[j].dcfclk_mhz <= clk_table->entries[i].dcfclk_mhz) {
				closest_clk_lvl = j;
				break;
			}
		}

		/* clk_table[1] is reserved for min DF PState.  skip here to fill in later. */
		if (i == 1)
			k++;

		dcn2_1_soc.clock_limits[k].state = k;
		dcn2_1_soc.clock_limits[k].dcfclk_mhz = clk_table->entries[i].dcfclk_mhz;
		dcn2_1_soc.clock_limits[k].fabricclk_mhz = clk_table->entries[i].fclk_mhz;
		dcn2_1_soc.clock_limits[k].socclk_mhz = clk_table->entries[i].socclk_mhz;
		dcn2_1_soc.clock_limits[k].dram_speed_mts = clk_table->entries[i].memclk_mhz * 2;

		dcn2_1_soc.clock_limits[k].dispclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dispclk_mhz;
		dcn2_1_soc.clock_limits[k].dppclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dppclk_mhz;
		dcn2_1_soc.clock_limits[k].dram_bw_per_chan_gbps = dcn2_1_soc.clock_limits[closest_clk_lvl].dram_bw_per_chan_gbps;
		dcn2_1_soc.clock_limits[k].dscclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dscclk_mhz;
		dcn2_1_soc.clock_limits[k].dtbclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dtbclk_mhz;
		dcn2_1_soc.clock_limits[k].phyclk_d18_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].phyclk_d18_mhz;
		dcn2_1_soc.clock_limits[k].phyclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].phyclk_mhz;

		k++;
	}
	if (clk_table->num_entries) {
		dcn2_1_soc.num_states = clk_table->num_entries + 1;
		/* fill in min DF PState */
		dcn2_1_soc.clock_limits[1] = construct_low_pstate_lvl(clk_table, closest_clk_lvl);
		/* duplicate last level */
		dcn2_1_soc.clock_limits[dcn2_1_soc.num_states] = dcn2_1_soc.clock_limits[dcn2_1_soc.num_states - 1];
		dcn2_1_soc.clock_limits[dcn2_1_soc.num_states].state = dcn2_1_soc.num_states;
	}

	dml_init_instance(&dc->dml, &dcn2_1_soc, &dcn2_1_ip, DML_PROJECT_DCN21);
}
```

This function has only two parameters:

* `struct dc *`: a pointer to a Display Core[^1] control structure;
* `struct clk_bw_params *`: a pointer to a structure for clocks and bandwidth.

After the declaration and initialization of variables, `dc_assert_fp_enabled` is
called to ensure that the function is being executed under FPU protection, as it
is about to perform some floating point operations. Then some values from the
global variable `dcn2_1_ip` are updated based on the provided parameters.
Finally, before the first loop begins, it is assured that the clock table from
`struct clk_bw_params` has at least one entry.

In the first loop, an odd thing happens:
```c
/* Copy dcn2_1_soc.clock_limits to clock_limits to avoid copying over null states later */
for (i = 0; i < dcn2_1_soc.num_states + 1; i++) {
    dcn2_1_soc.clock_limits[i] = dcn2_1_soc.clock_limits[i];
}
```
It seems like nothing meaninful is happening, since the elements of the clock
limits array are being set to themselves. At this point, it is useful to go back
and see how the function behaved before `8861c27a`:

```c
/* Copy dcn2_1_soc.clock_limits to clock_limits to avoid copying over null states later */
for (i = 0; i < dcn2_1_soc.num_states + 1; i++) {
	clock_limits[i] = dcn2_1_soc.clock_limits[i];
}
```

That makes more sense, and is coherent with the comment above it. It's important
to notice that commit `8861c27a` replaced occurences of `clock_limits` wit
`dcn2_1_soc.clock_limits`, so that is why the first for loop is like that.

## Finding the problem

Moving on, there's another loop. Notice its last lines:

```c
for (i = 0; i < clk_table->num_entries; i++) {
[...]
	dcn2_1_soc.clock_limits[k].dispclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dispclk_mhz;
	dcn2_1_soc.clock_limits[k].dppclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dppclk_mhz;
	dcn2_1_soc.clock_limits[k].dram_bw_per_chan_gbps = dcn2_1_soc.clock_limits[closest_clk_lvl].dram_bw_per_chan_gbps;
	dcn2_1_soc.clock_limits[k].dscclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dscclk_mhz;
	dcn2_1_soc.clock_limits[k].dtbclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dtbclk_mhz;
	dcn2_1_soc.clock_limits[k].phyclk_d18_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].phyclk_d18_mhz;
	dcn2_1_soc.clock_limits[k].phyclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].phyclk_mhz;

	k++;
}
```

Compare with the previous version, which used `clock_limits` to store the clock
limits values before assigning them to `dcn2_1_soc.clock_limits` directly:

```c
for (i = 0; i < clk_table->num_entries; i++) {
[...]
	clock_limits[k].dispclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dispclk_mhz;
	clock_limits[k].dppclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dppclk_mhz;
	clock_limits[k].dram_bw_per_chan_gbps = dcn2_1_soc.clock_limits[closest_clk_lvl].dram_bw_per_chan_gbps;
	clock_limits[k].dscclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dscclk_mhz;
	clock_limits[k].dtbclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].dtbclk_mhz;
	clock_limits[k].phyclk_d18_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].phyclk_d18_mhz;
	clock_limits[k].phyclk_mhz = dcn2_1_soc.clock_limits[closest_clk_lvl].phyclk_mhz;

	k++;
}
for (i = 0; i < clk_table->num_entries + 1; i++)
	dcn2_1_soc.clock_limits[i] = clock_limits[i];
```

Now to make it easier to visualize, suppose that before
`dcn21_update_bw_bounding_box` is called, some values from
`dcn2_1_soc.clock_limits` are as follows:

```c
dcn2_1_soc.clock_limits[2].dispclk = 757.89;
dcn2_1_soc.clock_limits[3].dispclk = 847.06;
dcn2_1_soc.clock_limits[4].dispclk = 900.00;
```

**Now after invoking `dcn21_update_bw_bounding_box`, what happens with the
values of `dcn2_1_soc.clock_limits[k].dispclk` for k = 3, closest_clk_lvl = 2;
and for k = 4, closest_clk_lvl = 3?**

* Previous version (8861c27a^): * `clock_limits[3].dispclk_mhz` is set to
	`dcn2_1_soc.clock_limits[2].dispclk_mhz`; * then
	`clock_limits[4].dispclk_mhz` is set to
	`dcn2_1_soc.clock_limits[3].dispclk_mhz`.
	
	After the end of the loop, `clock_limits` is copied onto
	`dcn2_1_soc.clock_limits`, resulting in:
```c
dcn2_1_soc.clock_limits[3].dispclk = 757.89;
dcn2_1_soc.clock_limits[4].dispclk = 847.06;
```

* Problematic version (8861c27a): * `dcn2_1_soc.clock_limits[3].dispclk_mhz` is
	set to `dcn2_1_soc.clock_limits[2].dispclk_mhz`; * then in the next
	iteration, `dcn2_1_soc.clock_limits[4].dispclk_mhz` is set to
	`dcn2_1_soc.clock_limits[3].dispclk_mhz`.

	After the end of the loop, this is the result:
```c
dcn2_1_soc.clock_limits[3].dispclk = 757.89;
dcn2_1_soc.clock_limits[4].dispclk = 757.89;
```

The part to pay attention here is that by dropping the intermediate variable,
`dcn2_1_soc.clock_limits` ends up being overwritten by itself, and some values
that it stored may be lost in the way.

## Building the test case

After understanding where the problem lays, it is time to write a test case that
covers it.

The function parameters will be as simple as possible, only the necessary struct
members will have their values set:

```c
.dc = {
	.res_pool = &(struct resource_pool) {
		.pipe_count = 2,
		.res_cap = &(struct resource_caps) {
			.num_timing_generator = 3
		},
	},
}
```

`bw_params.clk_table.entries[]` values will be set in a way so that a situation
where (k = 3, closest_clk_lvl = 2) and (k = 4, closest_clk_lvl = 3) will be
created, closely following what was discussed above.

```c
.bw_params {
	.num_channels = 1,
	.clk_table = {
		.entries = {
			{
				.voltage = 0,
				.dcfclk_mhz = 200,
				.fclk_mhz = 400,
				.memclk_mhz = 800,
				.socclk_mhz = 0,
			},
			{
				.voltage = 0,
				.dcfclk_mhz = 201,
				.fclk_mhz = 800,
				.memclk_mhz = 1600,
				.socclk_mhz = 0,
			},
			{
				.voltage = 0,
				.dcfclk_mhz = 553,
				.fclk_mhz = 1067,
				.memclk_mhz = 1067,
				.socclk_mhz = 0,
			},
			{
				.voltage = 0,
				.dcfclk_mhz = 602,
				.fclk_mhz = 1333,
				.memclk_mhz = 1600,
				.socclk_mhz = 0,
			},
			{
				.voltage = 0,
				.dispclk_mhz = 1372,
				.dppclk_mhz = 1372,
				.phyclk_mhz = 810,
				.phyclk_d18_mhz = 667,
				.dtbclk_mhz = 600,
			},
		},

		.num_entries = 5,
	}
}
```

`dcn2_1_soc` is a global variable, and its original values will be used for this
test case, so no need to set it differently.

After having the function input, the next step is building the expected values.
For this test case, `dcn2_1_soc` is the one expected to change. One easy way (is
it the right one, though?) to obtain the expected values is running the function
when it is in its correct state, and seeing what was yielded. Finally, one test
case is ready: it is now time to think about other ways to write others.

[^1]: [drm/amd/display - Display Core
    (DC)](https://www.kernel.org/doc/html/latest/gpu/amdgpu/display/index.html)
