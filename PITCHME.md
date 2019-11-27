---?color=linear-gradient(175deg, black 80%, silver 20%)

@snap[north span-100 text-13 text-pink]
A story about porting in-house 3D engine to Vulkan
@snapend

@snap[west span-40 text-blue text-bold]
Olgierd Humeńczuk
@snapend

@snap[east span-60 text-07 text-gray]
BIO
@ul[list-square-bullets text-silver](false)
- Has been programming for past 18 years using different programming languages
- Has created multiple game engines including software renderers
- Has been working on high performance networking servers and clients in C and C++
@ulend
@snapend

---?color=#444444

@snap[north span-100 text-white bg-orange text-08 text-bold]
What is rendering engine?
@snapend

@snap[east span-45 text-07 text-white text-center]
Responsible for:
@ul[list-square-bullets text-silver]
- preparing scene data to display
- managing resources such as: textures, geometry, shaders
- communication with GFX API
@ulend
@snapend

@snap[west span-45 text-07]
@box[bg-purple text-white](My definition # Rendering engine is a computer software with architecture similar to simple real time operating system that main task is to present given scene in a form of an image with stable framerate.)
@snapend

---?color=linear-gradient(90deg, #444444 50%, silver 50%)

@snap[north span-100 text-white text-08 text-bold bg-orange]
About OpenGL and Vulkan
@snapend

@snap[west span-49 text-white text-bold text-05 text-center]
OpenGL
@ul[list-square-bullets text-center]
- OpenGL 1.0 year 1992
- Originally architected for graphics workstations
- Driver does lots of work: state validation, dependency tracking, error checking
- Threading model doesn't enable generation of graphics commands in pararell to command execution
- Syntax evolved over twenty years - complex API choices can obscure optimal performance path
- Shader language compiler built into driver, Only GLSL supported. Have to ship shader source
- Debugging: get-last-error approach, verification during execution
@ulend
@snapend

@snap[east span-49 text-black text-bold text-05 text-center]
Vulkan
@ul[list-square-bullets text-center]
- Vulkan 1.0 years 2015/2016
- Matches architecture of modern platforms including mobile platforms
- Explicit API - the application has direct, predictable control over the operation of the GPU
- Multi-core friendly with multiple command buffers that can be created in pararell
- Removing legacy requirements simplifies API design. reduces specification size and enables clear usage guidance
- SPIR-V as compiler target simplifies driver and enables front-end language flexibility and reliability
- Debugging: multiple debug layers, verification through debug layers much more detailed information about origin of the error
@ulend
@snapend

---?color=#444444

@snap[span-100 text-white bg-orange text-08 text-bold]
Vulkan API
@snapend

![PLATE](assets/VulkanAPI.png)

---?color=#444444

@snap[north span-100 text-white bg-orange text-08 text-bold]
Vulkan entities we care
@snapend

@snap[east span-40 text-09]

@box[bg-green text-white text-09](Render Pass # Gathers setup for render targets and dependencies between them )

@box[bg-blue text-white text-09](Pipeline # Describes the configuration of input data, shaders etc.)

@snapend

@snap[west span-40 text-09]

@box[bg-orange text-white text-09](Queue # Three types of queues: Graphics, Compute, Transfer )

@box[bg-pink text-white text-09](Command Buffer # Gathers commands to be executed on any of the Queue )

@snapend

---?color=linear-gradient(90deg, #444444 50%, silver 50%)

@snap[north span-100 text-white bg-orange text-08 text-bold]
Programmer API comparision
@snapend

@snap[west span-49 text-white text-bold text-06 text-center]
OpenGL
@ul[list-square-bullets text-center]
- Functions executed without explicit context
- Functions ordering matter!
- Functions for managing resources
    - glCreate/glGen
- Functions for setting values
    - glClearColor(1.0f, 0.0f, 0.0f, 1.0f);
    - glUniform1f(loc, 0.234f);
@ulend
@snapend

@snap[east span-49 text-black text-bold text-06 text-center]
Vulkan
@ul[list-square-bullets text-center]
- Functions operates on three different levels
    - device level
    - command buffer level
    - queue level
- Functions for managing resources
    - vkCreateImage( device, ... );
- Functions for recording command buffer
    - vkCmdBindPipeline( cmd_buffer, ... );
- Functions for command buffer execution
    - vkQueueSubmit( queue, .... );
@ulend
@snapend


---?color=linear-gradient(90deg, #444444 50%, silver 50%)

@snap[span-100 text-white bg-orange text-13 text-bold]
Usage example
@snapend

@snap[span-100 text-white text-left text-bold text-08]
OpenGL
```cpp
glClearColor(1.0f, 0.0f, 0.0f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT);
```
@snapend

@snap[span-100 text-right text-black text-bold text-08]
Vulkan
```cpp
VkCommandBufferBeginInfo begin_info = {};

begin_info.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
begin_info.flags = VK_COMMAND_BUFFER_USAGE_SIMULTANEOUS_USE_BIT;

VkClearValue clear_value
    = {VkClearColorValue{1.0f, 0.0f, 0.0f, 1.0f}};

VkImageSubresourceRange image_range = {};
image_range.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
image_range.levelCount = 1;
image_range.layerCount = 1;

vkBeginCommandBuffer(cmd_buf, &begin_info);
vkCmdClearColorImage(cmd_buf, img,
    VK_IMAGE_LAYOUT_GENERAL, &clear_value, 1, &image_range);
vkEndCommandBuffer(cmd_buf);
```
@snapend

---?color=#444444
@snap[north span-100 text-white text-8 text-bold bg-orange]
Dilemma
@snapend

@snap[middlepoint span-100]
![PLATE](assets/fit.png)
@snapend

---?color=linear-gradient(90deg, #444444 50%, silver 50%)
@snap[north span-100 text-white text-08 text-bold bg-orange]
Let's build our "Abstraction" - let's treat OpenGL and Vulkan as simple state machines
@snapend

@snap[west span-50 text-white text-bold text-07]
OpenGL
![PLATE](assets/opengl_states.png)
@snapend

@snap[east span-50 text-black text-bold text-07]
Vulkan
![PLATE](assets/vulkan_states.png)
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
The "State Based" approach
@snapend

@snap[west span-50 text-white text-bold text-07]
@ul[list-square-bullets](false)
- Use OpenGL like API
- Each state change is registered in a temporary buffer
- Whenever the state changes fills all required fields or draw/flush encountered gather data and form Vulkan named states
- Verify if Vulkan named state already exist by using entity field hash function
@ulend
@snapend

@snap[east span-50]
![PLATE](assets/state_based_approach.png)
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
The "Command Buffer" approach
@snapend

@snap[west span-50 text-white text-bold text-07]
@ul[list-square-bullets](false)
- Use Vulkan like API
- Form Vulkan like named states
- Use command buffer to prepare graphics commands like in Vulkan
- OpenGL implementation can execute commands directly from the memory
@ulend
@snapend

@snap[east span-45]
![PLATE](assets/cmdbuf_based_approach.png)
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
Implementation - Abstract device
@snapend

@snap[middlepoint span-100 text-08]
```cpp
class abstract_device
{
public:
    virtual state_a_handle create_state_a(
        state_a_create_info&& ) = 0;
    virtual void release( state_a_handle sa ) = 0;

    virtual command_buffer
    create_command_buffer( const
        nv::gfx_cmds::gfx_cmd_union* data, size_t size ) = 0;
    virtual void
    update_command_buffer( command_buffer cb,
        const nv::gfx_cmds::gfx_cmd_union* data, size_t size ) = 0;
    virtual void release( command_buffer cb ) = 0;

    virtual void add_command_buffer_to_execution_queue(
        command_buffer cb ) = 0;
    virtual void submit_execution_queue( bool is_draw ) = 0;
    virtual void wait_for_idle();
};
```
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
Implementation - Command buffer
@snapend

@snap[middlepoint span-100 text-07]
```cpp
struct cmd_start_recording {};
struct cmd_end_recording {};
struct cmd_set_state_a { state_a_handle h; };

using gfx_cmd_union = safe_union< cmd_start_recording, cmd_end_recording,
    cmd_set_state_a >;

template < sint64 N > class gfx_cmds_recorder
{
public:
    using type = gfx_cmds::gfx_cmd_union;

    constexpr bool rec_cmd( type&& cmd )
    {
        new ( &m_data[m_current_size] ) type( nv::move( cmd ) );
        m_current_size += 1;
        return true;
    }
private:
    sint64 m_current_size;
    type* m_data;
};
```
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
Implementation - Execution OpenGL
@snapend

@snap[middlepoint span-100 text-07]
```cpp
struct gl_cmd_recorder
{
    template < typename T > void operator()( const T& /*cmd*/ )
    { NV_ASSERT_ALWAYS( false, "Unimplemented!" ); }
    inline void operator()( const nv::gfx_cmds::cmd_start_recording& /*cmd*/ ) { }
    inline void operator()( const nv::gfx_cmds::cmd_end_recording& /*cmd*/ ) { }
    inline void operator()( const nv::gfx_cmds::cmd_set_state_a& cmd )
    {
        const gl_set_state_a_info* state_a_info = m_data.get( cmd.h );
        glClearColor(state_a_info->color);
        ...
    }
    database& m_data;
};

void gl_device::add_command_buffer_to_execution_queue( command_buffer cb )
{
    // instant execution
    gl_command_buffer_info* info = m_command_buffers.get( cb );
    detail::gl_cmd_recorder recorder{m_data};
    for ( uint32 i = 0; i < info->count; ++i )
        visit( recorder, info->cmds[i] );
}
```
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
Summary
@snapend

@snap[middlepoint span-100 text-white text-bold text-05 text-center]
@ul[list-square-bullets text-center]
- Implementation of our engine bacame very explicit and straightforward
- Thanks to command buffer abstraction we are able to divide operations such as memory transfers from rendering, we can execute them separately
- We are able to render frames ahead or process different parts of frames simultaneously
- We've fixed many issues related to rendering that were hidden
- Performance of OpenGL got increased by 10% on average comparing to previous "direct" approach
@ulend
@snapend

---?color=#444444
@snap[north span-100 text-white text-08 text-bold bg-orange]
Bonus! Implementation - Visitor
@snapend

@snap[middlepoint span-100]
[Visitor Implementation](https://godbolt.org/z/VM_haa)
@snapend

---?color=#444444

@snap[north span-100 text-white text-08 text-bold bg-orange]
Thank you!
@snapend

Questions ?

