# Edge Deployment Checklist for Computer Vision

A practical checklist for deploying CV models to edge devices (NVIDIA Jetson, Google Coral, Intel NCS).

## Hardware Selection

| Device | Use Case | Inference Speed | Power | Price |
|--------|----------|----------------|-------|-------|
| Jetson Orin Nano | Mid-complexity models, multiple cameras | 40 TOPS | 15W | ~$500 |
| Jetson AGX Orin | Complex models, real-time video | 275 TOPS | 60W | ~$2000 |
| Google Coral | Lightweight classification/detection | 4 TOPS | 2W | ~$60 |
| Intel NCS2 | OpenVINO models, USB form factor | 1 TOPS | 1W | ~$70 |

## Model Optimization Pipeline

1. **Quantization**: FP32 → INT8 reduces model size 4x and speeds inference 2-3x
   ```bash
   # TensorRT quantization (Jetson)
   trtexec --onnx=model.onnx --int8 --saveEngine=model.trt
   
   # TFLite quantization (Coral)
   converter = tf.lite.TFLiteConverter.from_saved_model(model_path)
   converter.optimizations = [tf.lite.Optimize.DEFAULT]
   converter.target_spec.supported_types = [tf.int8]
   ```

2. **Pruning**: Remove low-importance weights. Typically 30-50% of weights can be pruned with <1% accuracy loss.

3. **Knowledge distillation**: Train a smaller "student" model to mimic a larger "teacher" model. Best when you need to hit a specific latency target.

## Deployment Checklist

### Pre-Deployment
- [ ] Model benchmarked on target hardware (not just desktop GPU)
- [ ] Inference latency meets SLA at p99 (not just average)
- [ ] Model fits in device memory with headroom for preprocessing
- [ ] Input preprocessing pipeline tested with real camera feeds
- [ ] Thermal testing done (edge devices throttle under sustained load)

### Connectivity & Sync
- [ ] Offline fallback mode implemented (what happens when cloud is unreachable?)
- [ ] Model update mechanism (OTA) tested and verified
- [ ] Telemetry data batched and compressed before upload
- [ ] Edge-to-cloud sync handles intermittent connectivity gracefully

### Monitoring
- [ ] Device health metrics exported (CPU/GPU temp, memory, disk)
- [ ] Inference metrics tracked locally (latency, throughput, errors)
- [ ] Anomaly detection on prediction distribution (local alert if drift detected)
- [ ] Remote management dashboard for fleet monitoring

### Security
- [ ] Model file encrypted at rest on device
- [ ] API/gRPC endpoint authenticated (even on local network)
- [ ] Firmware signing enabled for OTA updates
- [ ] Physical tamper detection if device is in public area

## Common Pitfalls

1. **Thermal throttling** — Edge devices in enclosed spaces (manufacturing floors, outdoor enclosures) overheat. Budget for active cooling or design duty cycles.

2. **Camera feed variability** — Models trained on clean datasets fail when cameras get dirty, lighting changes, or lens angles shift. Build preprocessing that normalizes these factors.

3. **Clock drift** — Edge devices without NTP sync will drift. This breaks timestamp-dependent logic and makes debugging impossible.

4. **Storage fills up** — Logging and image capture fill local storage fast. Implement rotation policies from day one.

---

*Part of the [Computer Vision Production Guide](../README.md) by [ShiftAI](https://shift-ai.cloud/computer-vision-solutions/).*
