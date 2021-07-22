# MicroProfiler Documentation

This documentation assumes you have a clear understanding and know how to use the MicroProfiler. To learn what MicroProfiler is, or how to use it, [read my article "Roblox: Using the MicroProfiler"](https://hackmd.io/@Mullets/H1_uG-FzO). The Roblox MicroProfiler, or profiler for short, is Roblox's built-in analysis tool that provides real-time, extremely accurate timings regarding your game's performance on the device used.

[ToC]

# Profiler Information

Useful profiler information that you should read and remember when analyzing and debugging your game.

## Notes
- Roblox frame budget is 17ms, 60 frames per second. This means your ideal frame should be much smaller than 17ms/frame.
- Networking runs in the 60hz pipeline, data received at the beginning of the frame & data sent at the end.
- "US" prefixed labels are security (example: `US14116`).
- Roblox plans to go past the 60 frame limit in the future for visuals, as far as the frame budget is concerned, that will remain the same.
- PreSimulation provides an accurate world space-time delta which is the same one the physics engine uses, more accurate than PostSimulation.
- Graphics quality heavily affects the GPU rendering & can lower workloads to compensate for any lag that might arise.
- Roblox uses Raknet, a networking library (UDP with a reliability layer on top):
    http://www.raknet.net/raknet/manual/introduction.html

## Keywords

Keywords can be found scattered around the labels. These are important to know when dealing with the profiler.

|Keyword|Description|
|-------|-----------|
|Mega|Internal optimization keyword, O(n)^2 -> O(n) for example.|
|$Script|A script running in Lua.|
|Contacts|2 unanchored parts or anchored part & unanchored part touching.|
|Task Scheduler (TS)|Run all code (jobs) concurrently.|
|TSMk2 Worker (thread)|CPU core/thread, most accurate timeline & information received will be located here. Helps main thread with networking, physics, and pathfinding. Multiple are used depending on the number of processor cores your machine has. TSMk2 is Roblox’s Task Scheduler name.|
|GPU [timeline]|A regular GPU device, but an inconsistent metric that should not be relied upon, but used for rough estimations.|
|DataModel Lock (DML)|DataModel lock is when a particular thread takes a mutex lock on the DataModel (the game instance) w/ exclusive read/write privileges. Only one thread can have a lock on a DataModel at a time, knowing where it's being held is useful for debugging where certain tasks happen. Scripts can run against the DataModel Lock concurrently.|

## Resources

- Official MicroProfiler article:
    https://developer.roblox.com/en-us/articles/MicroProfiler
- Official Performance article:
    https://developer.roblox.com/en-us/articles/Improving-Performance

# Profiler Labels

All of these labels can be found on the MicroProfiler. If they don't show up, they're either too fast & the profiler doesn't capture it, or the label doesn't exist and isn't running. Refer to [Profiler Information](#Profiler-Information) for help.

## GPU Timeline

### Scene:
The GPU timeline that renders everything (unreliable) including:

- [occupancyUpdateChunkPerform](#occupancyUpdateChunkPerform)
- [ID_Opaque](#ID_Opaque)
- [ID_Transparent](#ID_Transparent)
- [ID_Glass](#ID_Glass)
- [ID_Decals](#ID_Decals)
- [ResolveColor](#ResolveColor)
- [Linearizing depth to color target](#Linearize-Depth-to-color-target)
- [Sky](#Sky)
- [SSAO](#SSAO)
- [MSAA](#MSAA)
- [Glow, DoF, ColorCorrection](#Glow-DoF-ColorCorrection)

#### `occupancyUpdateChunkPerform:`
This specific label is pure shadows, find a way to disable shadows more if too high.

#### `ID_Opaque:`
All the parts rendered are opaque (transparency = 0).

#### `ID_Transparent:`
All the parts rendered are transparent or translucent (transparency != 0).

#### `ID_Glass:`
All the parts rendered are glass material.

#### `ID_Decals:`
All decals in the world.

#### `ResolveColor:`
Resolves MSAA main renders target into a plain texture. This one is required for certain post-effects (like DoF) and is done before any transparent objects are rendered.

#### `Linearize Depth to color target:`
Resolves and linearizes the MSAA depth buffer into a plain texture.

#### `Sky:`
Renders the skybox.

#### `SSAO:`
Applies screen-space ambient occlusion.

#### `MSAA:`
The main MSAA resolve step, same as the above Resolve_Color, but now includes everything, e.g. particles.

#### `Glow, DoF, ColorCorrection:`
Post-effects rendering.

## Primary CPU Thread Timeline

### Render:
CPU work the render job has to do. The second half of Render after PreRender ends is all SurfaceGui & BillboardGui rendering, which prove to be intensive & unoptimized in Roblox & takes about the same as PreRender’s time, making Render work x2 PreRender (Render takes 6ms, PreRender will take 3ms, and rendering world Gui will take 3ms). Communicates with the GPU of the device. Render also follows a prepare, perform, present logic:
- Prepare: Information from the main thread is used to update rendering models.
- Perform: Issue rendering commands, including 2D interfaces.
- Present: Synchronizes with the GPU.

Includes:
- [PreRender](#PreRender)
- [Worker::runJob](#WorkerrunJob)

#### `PreRender:`
Preparing the rendering by gathering all instances, kicks to the GPU to render. The [Scene](#Scene) label picks up from preparing, and PreRender includes:

- [ProcessInput](#ProcessInput)
- [fireBindToRenderStepCallbacks](#fireBindToRenderStepCallbacks)
- [RenderStepped](#RenderStepped)
- [tweenService](#tweenService)
- [Worker::runJob](#WorkerrunJob)

##### `ProcessInput:`
Fires user inputs such as `UIS.Changed`, `UIS.Began`, `UIS.Ended`.

##### `fireBindToRenderStepCallbacks:`
Runs all code hooked to `RunService:BindToRenderStep`, including:

- [ControlModule](#ControlModule)
- [CameraModule](#CameraModule)

###### `ControlModule:`
Runs Roblox controls here.

###### `CameraModule:`
Runs the popper cam function, surprisingly expensive to run, and runs other subtasks to the camera module.

##### `PreRender:`
Controls GetService, WalkSpeed, HipHeight, and `RunService.PreRender` functions, with:

- [tweenService](#tweenService)

##### `tweenService:`
Pretty optimized & lightweight.

#### `Worker::runJob:`
A thread that runs game jobs like LuaGC, Simulation, PostSimulation.

### LuaGC:
Overall garbage collection with:

- [GC](#GC)

#### `GC:`
Incremental garbage collection

### Simulation:
Processing the simulation on the CPU, which inclues:

- [gameStepped](#gameStepped)
- [physicsStepped](#physicsStepped)
- [interpolateNetworkedAssemblies](#interpolateNetworkedAssemblies)

#### `gameStepped:`
A complete step of dispatching animations & interpolating them along with updating humanoids in the game and includes:

- [stepAnimation](#stepAnimation)
- [RunService.Stepped]()
- [stepLegacy](#stepLegacy)

#### `stepLegacy:`
Step legacy updates any physics legacy objects such as body movers (body gyro, rocket propulsion), and includes:

- [Humanoid::onStepped](#HumanoidonStepped)

##### `Humanoid::onStepped:`
Humanoids (NPCs, Players) update under the here to track any humanoid performance in the world. Includes:

- [Humanoid::findForce](#HumanoidfindForce)

###### `Humanoid::findForce:`
Determines how much the humanoid should float off the floor, runs in 240hz physics pipeline.

#### `stepAnimation:`
Takes a DML, animators take the animation & dispatch it by interpolating parts, scripts can still run during this.

#### `physicsStepped:`
Crazy world math & generates a simulation, physics info: 240hz, PGS solver, take the current state of the world 4 times to achieve the goal, includes:

- [worldStep](#worldStep)

##### `worldStep:`
Physics side optimizations planned to decrease how many worldSteps exist will benefit greatly (physics solver won’t be doing as much work). Iteration of physicsStepped, has a single PGS step, applies forces, discrete unit, some physics work happens, the engine gets closer to a solution, will reduce to run at 240hz. If `worldStep` goes above 3ms, the engine triggers “throttling” which skips physics work for any objects that aren’t considered time-critical, like player characters. This label includes:

- [updateBroadphase](#updateBroadphase)
- [newStepContacts](#newStepContacts)

###### `updateBroadphase:`
Updates the lighting broad phase for things like shadows, is extremely expensive so try to reduce moving parts and decreasing shadows on parts by only enabling `.CastShadow` on parts that require shadows.

###### `newStepContacts:`
Introduces a new step contact, try to decrease this label by reducing contacts. Also includes:

- [preContactStepSleepStage](#preContactStepSleepStage)
- [stepContacts](#stepContacts)

###### *`preContactStepSleepStage:`*
Unknown; presumably determining if a touch should go to sleep before starting a new touch.

###### *`stepContacts:`*
Every touch between contact is simulated & determined. Uses just Narrow phase collision detection. Enable “Are Contact Points Shown” to help visualize. If the timeline is lengthy, look for CollisionFidelity.Default, make more things CanCollide false and reduce the number of parts (including welded parts).

#### `jointStepUi:`
Mostly Motor6D transforms being applied.

#### `Solver::solve:`
If this label is long, try reducing constraints or collisions.

#### `interpolateNetworkedAssemblies:`
Takes everything not owned by you (NetworkOwnership). Sent to the player at an uneven interval, interpolates the network replication from the last point they were to interpolate. Makes games feel smooth, and the average time spent here is 1ms.

### PostSimulation:
- [postsimulationInternal](#postsimulationInternal)
- [RunService.PostSimulation](#RunServicePostSimulation)

#### `postsimulationInternal:`
Roblox’s internal PostSimulation which executes before `RunService.PostSimulation`.

#### `RunService.PostSimulation:`
Fires all functions connected to `RunService.PostSimulation`.

### Legacy:
These are both very crusty mechanisms that we are trying to use less of, please report high amounts of on the Dev Forums under ‘Studio Bugs’, which include:

- [Write Marshalled](#Write-Marshalled)
- [Read Marshalled](#Read-Marshalled)
- [LegacyLock](#LegacyLock)

#### `Write Marshalled:`
An exclusive write job that stops the world and takes control of the DataModel.

#### `Read Marshalled:`
An exclusive read job that stops the world and takes control of the DataModel.

#### `LegacyLock:`
An exclusive read/write job that stops the world and takes control of the DataModel.

### Networking:
The bridge for communication between the client and server, which includes:

- [HttpRbxApiJob](#HttpRbxApiJob)
- [Replicator Process Packets](#Replicator-Process-Packets)
- [SendData Mega Job](#SendData-Mega-Job)
- [eplicator SendData](#Replicator-SendData)

#### `HttpRbxApiJob:`
This is when Roblox passes all Roblox-based Http tasks such as retrieving an avatar image. Non-Roblox Http is not included here (need confirmation).

#### `Replicator Process Packets:`
This is a 60hz process, taking data from both server & client (depending on if you're on the server or client). This label takes a DML & takes all changes to replicate & sends it along with remote events & functions. This label is specifically for inbound replication. This label must process the entire world when receiving packets, so things like lots of players will balloon up this process.

#### `SendData Mega Job:`
This job sends at the end of each frame, so remotes & other requests will be queued to send. This is typically not the source of any lag, but be wary of this label is large. Queued changes send to the server or client using Raknet.

#### `Replicator SendData:`
This replicator process is what sends the process data to the network stack. From there, the network stack puts it in the computer’s networking drivers (Roblox does not utilize their own driver method - using an OS is easier & lightweight) & sends it out to either the client or server respectively.

## Secondary CPU Thread Timeline
The subtasks thread that runs other jobs, including:

- [Prepare](#Prepare)
- [Perform](#Perform)

### Prepare:
Takes a DML and collects all instances & passes them to the GPU to render. This task includes Gui & BaseParts, and the collection process puts all of these instances into a special hash that the GPU can read. This includes:

- [Pass3dAdorn](#Pass3dAdorn)
- [Pass2dAdorn](#Pass2dAdorn)

#### `Pass3dAdorn:`
Passes Interface to world Gui such as Billboard & Surface Gui (3d UI). This is expensive & can commonly be a culprit of performance issues.

#### `Pass2dAdorn:`
Passes interface to ScreenGui (2d UI).

### Perform:
Sends the data from Prepare to the GPU to perform the rendering, GPU based profiling occurs here. Dispatch all instructions to GPU, do note that this is not on the main thread.

---

Thanks for reading, follow me on Twitter [@Mullets_Gavin](https://twitter.com/Mullets_Gavin).