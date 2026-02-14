
# Table of Contents

1.  [PendSV Handler](#orgbeeb68e)
    1.  [Controlling the Handler](#org1ecc3d3)
2.  [SysTick Handler](#orga77429d)
    1.  [Controlling the Handler](#org16614d3)
3.  [SVCall](#org3d31934)
    1.  [Controlling the Handler](#org3ce9d1a)
4.  [Critical Error in `os_cpu.h`](#orgebf672c)



<a id="orgbeeb68e"></a>

# PendSV Handler

This is the heart of task control on an RTOS. It is always set to the lowest priority, so that it only blocks user threads, and no interrupts are running in the background while a context switch is performed.

Context switch: stack (push/save to TCB on stack) running thread &rarr; unstack (pop/restore) ready thread.

Unstack:

-   general-purpose registers
-   stack pointer (PSP)
-   "BX LR": PC &larr; LR

Only accessible from **handler mode**.


<a id="org1ecc3d3"></a>

## Controlling the Handler

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Intent</th>
<th scope="col" class="org-left">Action</th>
</tr>
</thead>
<tbody>
<tr>
<td class="org-left">Set priority to lowest</td>
<td class="org-left">SCB-&gt;SHP[10] = 0xF0</td>
</tr>

<tr>
<td class="org-left">Trigger exception</td>
<td class="org-left">SCB-&gt;ICSR &vert;= (1 &lt;&lt; 28)</td>
</tr>
</tbody>
</table>


<a id="orga77429d"></a>

# SysTick Handler

This is usually also set to the lowest priority. On an RTOS, it is used to implement a round-robin scheduler, or a rate-monotonic effect between tasks.

Calls PendSV every wake up.


<a id="org16614d3"></a>

## Controlling the Handler

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Intent</th>
<th scope="col" class="org-left">Action</th>
</tr>
</thead>
<tbody>
<tr>
<td class="org-left">Set priority to lowest</td>
<td class="org-left">SCB-&gt;SHP[11] = 0xF0</td>
</tr>

<tr>
<td class="org-left">Trigger exception</td>
<td class="org-left">SCB-&gt;ICSR &vert;= (1 &lt;&lt; 26)</td>
</tr>
</tbody>
</table>


<a id="org3d31934"></a>

# SVCall

Used to trigger a system service (e.g. send messages, notify threads, etc.). Acts as a "bridge" between **thread mode** or **interrupts**, and **privileged mode**. It is usually set to the highest priority (below system handlers), so that it can run immediately.

Can trigger PendSV if necessary. Used to start the first task in FreeRTOS.


<a id="org3ce9d1a"></a>

## Controlling the Handler

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left">Intent</th>
<th scope="col" class="org-left">Action</th>
</tr>
</thead>
<tbody>
<tr>
<td class="org-left">Set priority to just above lowest</td>
<td class="org-left">SCB-&gt;SHP[7] = 0x10</td>
</tr>

<tr>
<td class="org-left">Trigger exception</td>
<td class="org-left">asm("svc &lt;number&gt;")</td>
</tr>
</tbody>
</table>


<a id="orgebf672c"></a>

# Critical Error in `os_cpu.h`

Line 17-18:

    /* Make PendSV and SysTick the lowest priority interrupts. */
    #define os_cpu_setup_PendSV() \
        (*(uint32_t volatile *)0xE000ED20 |= (0xFFU << 16))

As mentioned in section [PendSV Handler](#orgbeeb68e) and [SysTick Handler](#orga77429d), each handler's priority is specified in a distinct byte. So to "make PendSV and SysTick the lowest priority interrupts", we can do it in either way:

1.  As in the mentioned sections sequentially.
2.  `#define os_cpu_setup_PendSV() ((volatile uint32_t *)(SCB->SHP)[3] |= (0xFFFF << 8))`

