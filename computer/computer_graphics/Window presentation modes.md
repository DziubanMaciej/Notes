parentTopic:: [[Computer graphics]]
links:: https://www.youtube.com/watch?v=E3wTajGZOsA&t=1s

# Presentation modes in Windows 10
These notes are mainly derived from [https://www.youtube.com/watch?v=E3wTajGZOsA](this presentation) by Jesse Natalie. They are also suplemented by some materials available online.

Modes
1. Windowed BLT - old presentation mode from D3D9. App renders to two buffers (alternating between them). During *present* call the runtime copies contents to the actual presentable surface. Then compositor (DWM) will copy it once more. To trigger it, create swapchain with DXGI_SWAP_EFFECT_DISCARD. It cannot by used in D3D12.
2. Fullscreen flip - two buffers. One is on screen and the second one is rendered to. After *present* call they are flipped. To trigger it, create swapchaing with or DXGI_SWAP_EFFECT_FLIP_SEQUENTAIL or DXGI_SWAP_EFFECT_FLIP_DISCARD.
3. Fullscreen blit - same as Windowed BLT, but without a compositor.
4. Fullscreen proxy (flip+blit) - app renders to two buffers (alternating between them). Runtime has 2 additional presentable buffers. During *present* call the runtime copies buffers rendered by the app to presentable buffers, which are flipped. This mode is used automatically by the OS, if for some reason your buffers are not capable of being on screen. Usually that would happen because of some incorrect settings during swapchain creation, e.g. incompatible refresh rate or scan line ordering.
5. Windowed flip - two back buffers, which app renders to are shared with the compositor, so there is no need to perform a copy. Introduced in Windows 7.
6. Windowed direct flip (dFlip) - compositor can detect that the windows covers entire screen and decide to completely skip composing. Introduced in Windows 8.
7. Windowed independent flip (iFlip) -  Introduced in Windows 8.1.
8. Windowed true immediate independent flip. Used only for D3D12 to implement exclusive fullscreen APIs.

Different presentation modes have different characteristics of frame latency. High latency can be bad for input critical applications (e.g. action video games), but useful to hide hiccups.

Blit and flip models have their **_DISCARD** flavors. These modes give no guarantee that the contents of the backbuffer after presenting will be the same. The compositor (DWM) can use it to render some UI overlays (e.g. volume overlay) on top of application's contents instead of copying to some other buffer.

FPS of windowed apps (which are not fullscreen and not use iFlip/dFlip) doesn't matter, because they have to be composited anyway, and the compositor works at monitor refresh rate.

Multiplane overlay (MPO)
