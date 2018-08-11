LV2 Audio-Plugin Workshop OSAMC
################################

:author: Christopher Arndt
:date: 2018-08-11


Let's create a plug-in, which generates some audio output itself. It will output a sawtooth wave,
whose frequency can be controlled via a plug-in parameter. We'll call the plug-in "SimpleSaw".

We can simply generate the same project as for the `SimpleGain plug-in`_ and then change the
source code in a few places to make the plug-in do something different. You can see all the changes
in detail in `this commit`_ in the GitHub repository for the plug-in.

.. _simplegain plug-in: my-first-lv2-plugin.rst
.. _this commit:
    https://github.com/osamc-lv2-workshop/simplesaw/commit/b5750ecba67ade452dee86775aafd4228f0f4205


Simple Saw Plugin
-----------------

1. Initialize a new DSP audio plug-in `project with cookiecutter <my-first-lv2-plugin.rst>`_ and
   name it ``SimpleSaw``.

2. Copy the following files from `this repository`_ into the ``plugins/SimpleSaw`` directory:

   * ``WaveTableOsc.{c,h}pp``
   * ``WaveUtils.{c,h}pp``

   If you download the files via the GitHub web interface, make sure you get the actual raw files,
   not the HTML pages displaying the source code of the files (use the "Raw" button on the source
   disply page of each file).

3. Add these objects to the ``OBJ_DSP`` list in the Makefile in ``plugins/SimpleSaw`` *before*
   ``PluginSimpleSaw.cpp.o`` (don't forget the backslashes to keep the whole assigment on one
   logical line)::

        WaveTableOsc.cpp.o \
        WaveUtils.cpp.o \

4. Add ``#include "WaveUtils.hpp"`` to ``plugins/SimpleSaw/PluginSimpleSaw.hpp`` after the
   ``#include`` that's already in there.

5. Add ``WaveTableOsc *osc;`` to the ``private`` Section of the ``PluginSimpleSaw`` class in
   ``plugins/SimpleSaw/PluginSimpleSaw.hpp`` and remove the ``double   fSampleRate;`` line there.

6. Add ``osc = sawOsc();`` in the constructor method of the ``PluginSimpleSaw`` class in
   ``plugins/SimpleSaw/PluginSimpleSaw.cpp`` *before* the ``loadProgram(0);`` line and remove
   the ``sampleRateChanged(getSampleRate());`` line.

7. Change every occurence of ``paramVolume`` to ``paramFrequency`` in
   ``plugins/SimpleSaw/PluginSimpleSaw.hpp`` and ``plugins/SimpleSaw/PluginSimpleSaw.cpp``.

8. Change the ``initParameter`` method of the ``PluginSimpleSaw`` class to set the name and symbol
   of the parameter to "Frequency" resp. "frequency" and its min/max/default values to ``20.0`` /
   ``10000.0`` and ``440.0``;

9. Change the for loop in the ``run`` method of the ``PluginSimpleSaw`` class to the following::

        // output the oscillator waveform
        for (uint32_t i=0; i < frames; ++i) {
            outL[i] = osc->getOutput() * 0.8;
            outR[i] = osc->getOutput() * 0.8;
            osc->updatePhase();
        }

   and remove the lines initializing  the ``inpL``, ``inpR`` and ``vol`` variables.

   Optionally, remove the ``input`` name from the method parameters, since the plug-in doesn't use
   the audio inputs.

10. Change the body of the ``setParametervalue`` method of the ``PluginSimpleSaw`` class to the
    following::

        fParams[index] = value;

        switch (index) {
            case paramFrequency:
                osc->setFrequency(value / getSampleRate());
                break;
        }

11. Change the body of the ``loadProgram`` method of the ``PluginSimpleSaw`` class to the
    following::

        switch (index) {
            case 0:
                setParameterValue(paramFrequency, 440.0f);
                break;
        }

    (Only the second parameter in the call to ``setParameterValue`` changes.)

12. Change the body of the ``sampleRateChanged`` method of the `PluginSimpleSaw`` class to the
    following single line::

        osc->setFrequency(fParams[paramFrequency] / newSampleRate);

13. Set the value of the ``DISTRHO_PLUGIN_NUM_INPUTS`` pre-processor definition in
    ``DistrhoPluginInfo.h`` to ``0``.

14. Compile the plug-in, copy or symlink the LV2 plug-in to ``~/.lv2`` and test it with ``jalv``
    or another LV2 host of your liking.


.. _this repository:
    https://github.com/SpotlightKid/MonoSynth/tree/master/plugins/monosynth
